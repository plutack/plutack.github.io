---
title: "Seedrlike: A Three-Month Development Journey and Retrospective"
date: 2025-06-03T10:31:10+01:00
draft: false
---


For the past three months, I've been intermittently developing [Seedrlike]({{< relref "seedr-alternative-in-go-2025-02-16.md" >}}). Before diving into new features, I felt it was a good time to pause and reflect on what I've built, the challenges I encountered, and the lessons learned along the way.

## Initial Forays into Go

When I embarked on building seedrlike, Go was still relatively new territory for me. Navigating its syntax, unique features like embedding via composition (as opposed to traditional inheritance), and its idiomatic error handling presented a learning curve. However, over these past few weeks, I've definitely felt a significant improvement in my proficiency and comfort with the language.

## Seedrlike: Core Functionality and Evolution

On the surface, seedrlike is a straightforward web application. Its primary domain logic revolves around processing torrent links. Initially, these links were added to a Go channel. As the design evolved, this was refactored to use a map. A global worker service would monitor this collection for new additions and handle the downloads. In the early stages (when it was an array/slice, before the map), this worker operated as an infinite loop, processing tasks synchronously, one by one. After a torrent is downloaded, the application also handles zipping the folder if the user requested it.

A key external dependency is Gofile, my chosen third-party storage solution. Gofile has a well-documented API, but to streamline its integration into seedrlike, I took a detour to create a Go API wrapper. This side project was also an excellent opportunity to practice writing a reusable Go package, adhering strictly to semantic versioning (semver) principles.

## Reaching an MVP and Tackling New Features

Eventually, seedrlike reached its first usable state (Minimum Viable Product). However, as I started to add more features, I encountered new challenges, particularly around providing better feedback to the user.

The seedrlike interface initially only displayed backend activity during the file downloading phase. But a task can exist in several states:

    Pending: The initial state for every task after a link is added. Torrent metadata is unknown at this point.

    Downloading: The state after torrent metadata is fetched and all necessary information is populated.

    Zipping: If the user specifies, this state follows downloading.

    Uploading: The state where the downloaded (and possibly zipped) file is being uploaded to a Gofile server.

    Intermediate States: There are also brief transitional states between these primary ones.

As mentioned, the frontend initially only provided feedback (progress bar, percentage completion, ETA) during the "Downloading" state. Once that completed, the user had no visibility into subsequent processes like zipping or uploading, as no websocket updates were sent during these stages.
Enhancing Real-time Feedback: Websockets and Callbacks

To implement comprehensive real-time updates, I needed to extend websocket announcements to cover the zipping and uploading stages.

1. **Upload Progress**: This involved modifying the function signature of the UploadFile method in my Gofile API wrapper. The goal was to enable it to accept a callback function. This callback would receive the total size of the file and the number of bytes uploaded so far, allowing me to simulate progress updates and propagate them back to the frontend via websockets.

2. **Zipping Progress**: For the zipping stage, Go's interface system and the ability to wrap existing io.Reader implementations proved invaluable. The io.Reader interface has a Read method. By creating a custom struct that embeds an io.Reader and implements its Read method, I could intercept each chunk read during the zipping process. This custom Read method then triggers a websocket update.

Initially, managing the websocket connection logic led to a cyclic import error, which was flagged. I resolved this by creating a dedicated websocket manager and carefully structuring its integration.

Here's a look at the evolution of the zipping logic:

Zipping: Before

```go
// Original call to ZipFolder
if err = upload.ZipFolder(originalPath, zipPath); err != nil {
    // Handle error
}
// Original ZipFolder function
func ZipFolder(source string, destination string) error {
    zipFile, err := os.Create(destination)
    if err != nil {
        return err
    }
    defer zipFile.Close()

    zipWriter := zip.NewWriter(zipFile)
    defer zipWriter.Close()

    return filepath.Walk(source, func(path string, info os.FileInfo, err error) error {
        // ... (error checking and header creation) ...
        header, err := zip.FileInfoHeader(info)
        if err != nil {
            return err
        }
        header.Name, _ = filepath.Rel(filepath.Dir(source), path) // Simplified for brevity
        header.Method = zip.Deflate
        if info.IsDir() {
            header.Name += "/"
        }

        zipEntry, err := zipWriter.CreateHeader(header)
        if err != nil {
            return err
        }

        if info.IsDir() {
            return nil
        }

        srcFile, err := os.Open(path)
        if err != nil {
            return err
        }
        defer srcFile.Close()

        _, err = io.Copy(zipEntry, srcFile) // Standard io.Copy, no progress
        return err
    })
}
```
Zipping: After (with Progress Tracking)
The calling code was updated to pass a progress callback:

```go
// Updated call to ZipFolder, now with a progress callback
calculateZipProgress := func(readByte, totalByte int64) {
    var progress float64 = 0
    if totalByte > 0 {
        progress = float64(readByte) * 100 / float64(totalByte)
    }
    progress = math.Round(progress*100) / 100 // Round to 2 decimal places

    wm.SendProgress(ws.TorrentUpdate{ // wm is the websocket manager
        Type:     "torrent update",
        ID:       infoHash,
        Name:     t.Info().Name,
        Status:   StatusZipping,
        Progress: progress,
        Speed:    "-",
        ETA:      "--:--",
    })
}

if err = upload.ZipFolder(originalPath, zipPath, wm, calculateZipProgress); err != nil {
    // Handle error
}

And the ZipFolder function was enhanced:

// Type for the progress callback function
type progressCallbackFunc func(byteRead, totalBytes int64)

// readerProgress wraps an io.Reader to track read progress
type readerProgress struct {
    io.Reader // Embedded reader
    totalRead *int64 // Pointer to a running total of bytes read across all files
    totalSize int64 // Total size of the source folder/file being zipped
    onRead    progressCallbackFunc // Callback to execute on each Read operation
}

// Read implements io.Reader and updates progress
func (rp *readerProgress) Read(p []byte) (n int, err error) {
    n, err = rp.Reader.Read(p) // Call the embedded reader's Read method
    if n > 0 {
        *rp.totalRead += int64(n) // Update the running total
        if rp.onRead != nil {
            // Execute the callback with current progress for the entire zip operation
            rp.onRead(*rp.totalRead, rp.totalSize)
        }
    }
    return
}

// Helper function (conceptual) to get total size of a directory
func getPathSize(path string) (int64, error) {
    var size int64
    err := filepath.Walk(path, func(_ string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        if !info.IsDir() {
            size += info.Size()
        }
        return nil
    })
    return size, err
}


// Updated ZipFolder function signature now includes websocket manager and callback
func ZipFolder(source string, destination string, wm *ws.WebsocketManager, progressCallback progressCallbackFunc) error {
    zipFile, err := os.Create(destination)
    if err != nil {
        return err
    }
    defer zipFile.Close()

    // Calculate total size of the source directory for accurate progress
    totalSize, err := getPathSize(source)
    if err != nil {
        return fmt.Errorf("failed to calculate folder size: %w", err)
    }

    zipWriter := zip.NewWriter(zipFile)
    defer zipWriter.Close()

    var totalRead int64 // Tracks total bytes read across all files for the zip operation

    return filepath.Walk(source, func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        // ... (header creation logic remains similar) ...
        header, err := zip.FileInfoHeader(info)
        if err != nil {
            return err
        }
        header.Name, _ = filepath.Rel(filepath.Dir(source), path) // Simplified
        header.Method = zip.Deflate
        if info.IsDir() {
            header.Name += "/"
        }
        zipEntry, err := zipWriter.CreateHeader(header)
        if err != nil {
            return err
        }
        if info.IsDir() {
            return nil
        }

        srcFile, err := os.Open(path)
        if err != nil {
            return err
        }
        defer srcFile.Close()

        // Wrap the source file reader with our progress tracking reader
        srcFileWithProgress := &readerProgress{
            Reader:    srcFile,
            totalRead: &totalRead,          // Pass the pointer to the running total
            totalSize: totalSize,           // Pass the total size of the entire source
            onRead:    progressCallback,    // The callback to invoke
        }

        // io.Copy will now use our custom Read method, triggering progress updates
        _, err = io.Copy(zipEntry, srcFileWithProgress)
        return err
    })
}
```

The srcFileWithProgress instance ensures that its onRead function (our calculateZipProgress callback) is invoked whenever its internal Read method is called by io.Copy. This happens for each chunk read from the source file, allowing for granular progress updates for the zipping process.

A similar approach was taken for the file upload stage. To avoid the cyclic import error with the websocket manager, I embedded the manager within a new ProgressTrackingUploader struct.

File Upload: Before

```go
// Original uploadFile function
func uploadFile(fullFilePath string, parentFolderID string, rootFolderID string, uploadClient *api.Api, db *database.Queries, server string) error {
    info, err := uploadClient.UploadFile(server, fullFilePath, parentFolderID) // No progress callback
    if err != nil {
        return err
    }
    // ... (database update logic) ...
    folderID := info.Data.ParentFolder
    if folderID == rootFolderID {
        folderID = RootFolderPlaceholder
    }
    fileDetails := database.CreateFileParams{
        ID:       info.Data.ID,
        Name:     info.Data.Name,
        FolderID: folderID,
        Size:     info.Data.Size,
        Mimetype: info.Data.Mimetype,
        Md5:      info.Data.MD5,
        Server:   info.Data.Servers[0],
    }
    if err := db.CreateFile(context.Background(), fileDetails); err != nil {
        return err
    }
    uploadClient.UpdateContent(info.Data.ParentFolder, "public", true)
    return nil
}
```

File Upload: After (with Progress Tracking)
A new struct ProgressTrackingUploader was introduced:

```go
// ProgressTrackingUploader handles upload progress reporting
type ProgressTrackingUploader struct {
    client          *api.Api             // The Gofile API client
    websocketMgr    *ws.WebsocketManager // Embedded websocket manager
    torrentID       string               // InfoHash of the torrent
    torrentName     string               // Name of the torrent
    totalBytes      int64                // Total bytes to upload
    totalUploaded   int64                // Running total of bytes uploaded
    progressPercent float64              // Current progress percentage (to avoid too frequent updates)
}

// NewProgressTrackingUploader creates a new upload tracker
func NewProgressTrackingUploader(client *api.Api, wm *ws.WebsocketManager, id string, name string, totalSize int64) *ProgressTrackingUploader {
    return &ProgressTrackingUploader{
        client:          client,
        websocketMgr:    wm,
        torrentID:       id,
        torrentName:     name,
        totalBytes:      totalSize,
        totalUploaded:   0,
        progressPercent: 0.0,
    }
}

// UpdateProgress is the callback function that reports upload progress
func (u *ProgressTrackingUploader) UpdateProgress(bytesUploadedChunk int64) { // Renamed for clarity
    u.totalUploaded += bytesUploadedChunk
    if u.totalBytes > 0 {
        newProgress := float64(u.totalUploaded) * 100 / float64(u.totalBytes)
        // Only send update if progress changed by at least 1%
        if int(newProgress) > int(u.progressPercent) {
            u.progressPercent = newProgress
            roundedProgress := math.Round(newProgress*100) / 100

            u.websocketMgr.SendProgress(ws.TorrentUpdate{
                Type:     "torrent update",
                ID:       u.torrentID,
                Name:     u.torrentName,
                Status:   "uploading",
                Progress: roundedProgress,
                Speed:    "-", 
                ETA:      "--:--", 
            })
        }
    }
}

// The uploadFile function signature is updated to accept the progress callback
// (In this setup, the callback is a method of ProgressTrackingUploader)
// The actual call to uploadClient.UploadFile would now need to accept this callback.
func uploadFile(fullFilePath string, parentFolderID string, rootFolderID string,
    ptu *ProgressTrackingUploader, // Pass the ProgressTrackingUploader instance
    db *database.Queries, server string) error { // uploadClient is now ptu.client

    // The Gofile API client's UploadFile method now needs to accept and use the callback
    // The callback to pass to the underlying Gofile API client's method would be ptu.UpdateProgress
    info, err := ptu.client.UploadFile(server, fullFilePath, parentFolderID, ptu.UpdateProgress)
    if err != nil {
        return err
    }
    // ... (database update logic remains similar) ...
    folderID := info.Data.ParentFolder
    if folderID == rootFolderID {
        folderID = RootFolderPlaceholder
    }
    fileDetails := database.CreateFileParams{
        ID:       info.Data.ID,
        Name:     info.Data.Name,
        FolderID: folderID,
        Size:     info.Data.Size,
        Mimetype: info.Data.Mimetype,
        Md5:      info.Data.MD5,
        Server:   info.Data.Servers[0],
    }
    if err := db.CreateFile(context.Background(), fileDetails); err != nil {
        return err
    }
    ptu.client.UpdateContent(info.Data.ParentFolder, "public", true)
    return nil
}
```

The gofile dependency's internal upload function (which is called by uploadClient.UploadFile) is responsible for creating an io.PipeReader and using a custom progressReader (similar to the one for zipping) that invokes the provided onProgress callback as data is read from the file and written to the multipart request body.
```go
// Snippet from the Gofile API wrapper dependency (illustrative)
 type ProgressCallback func(bytesRead int64) 

type progressReader struct {
    io.Reader
    size   int64
    total  int64
    onRead ProgressCallback // This is the callback like ptu.UpdateProgress
}

func (pr *progressReader) Read(p []byte) (n int, err error) {
    n, err = pr.Reader.Read(p)
    if n > 0 {
        pr.total += int64(n)
        if pr.onRead != nil {
            pr.onRead(int64(n)) // Call the progress callback with bytes read in this chunk
        }
    }
    return
}

// Simplified internal upload function in the Gofile client
func (c *Client) upload(filePath string, folderId string, contentType *string, onProgress ProgressCallback) *io.PipeReader {
    pr, pw := io.Pipe() // Pipe reader and writer
    formWriter := multipart.NewWriter(pw) // Multipart writer

    go func() { // Goroutine to handle writing to the pipe
        defer pw.Close() // Ensure pipe writer is closed
        defer formWriter.Close() // Ensure multipart writer is closed

        // ... (write other form fields) ...
        err := formWriter.WriteField("folderId", folderId)
        if err != nil {
            pw.CloseWithError(err)
            return
        }

        file, err := os.Open(filePath)
        if err != nil {
            pw.CloseWithError(err)
            return
        }
        defer file.Close()

        fileInfo, err := file.Stat()
        if err != nil {
            pw.CloseWithError(err)
            return
        }

        // Wrap the file reader with progressReader
        progressR := &progressReader{
            Reader: file,
            size:   fileInfo.Size(),
            total:  0,
            onRead: onProgress, // This is where ptu.UpdateProgress gets hooked in
        }

        part, err := formWriter.CreateFormFile("file", fileInfo.Name())
        if err != nil {
            pw.CloseWithError(err)
            return
        }

        // io.Copy will read from progressR, triggering onRead callback
        if _, err = io.Copy(part, progressR); err != nil {
            pw.CloseWithError(err)
            return
        }
    }()

    *contentType = formWriter.FormDataContentType()
    return pr // Return the pipe reader for the HTTP request body
}
```


Gofile client's UploadFile method except from the third party dependency
```go
func (c *Client) UploadFile(server string, filePath string, folderID string, callbackUpdate ProgressCallback) (*UploadResponseInfo, error) {
    // Changed return type for clarity
    uploadURL := getUploadServerURL(server)
    var requestContentType string
    
    // The 'upload' function sets up the piped reader with progress tracking
    pipeReader := c.upload(filePath, folderID, &requestContentType, callbackUpdate)

    // No timeout during the upload itself, as it's handled by the pipe
    // c.httpClient.Timeout = 0 // This will be replaced by context

    req, err := http.NewRequest(http.MethodPost, uploadURL, pipeReader)
    if err != nil {
        return nil, err
    }
    setAuthorizationHeader(req, c.config.APIToken)
    req.Header.Set("Content-Type", requestContentType)

    // Use a context-aware HTTP client or request for better timeout/cancellation
    // For now, assuming the original c.httpClient.Do(req)
    response, err := c.httpClient.Do(req) // This is where the actual upload happens
    if err != nil {
        return nil, err
    }
    // c.httpClient.Timeout = 1 * time.Minute // Reset timeout, also to be replaced

    // ... (process response and return UploadResponseInfo) ...
    // This is a placeholder for actual response processing
    var uploadInfo UploadResponseInfo 
    if err := json.NewDecoder(response.Body).Decode(&uploadInfo); err != nil {
        response.Body.Close()
        return nil, err
    }
    response.Body.Close()
    return &uploadInfo, nil
}
```

## Moving Forward: Planned Improvements

Before I consider seedrlike "complete" for its current scope, I'd like to address a few areas:

1. **Context-Aware Requests**: Having gained more insight into Go's context package, I see opportunities to improve request handling, particularly for HTTP calls. Currently, my Gofile API client's `UploadFile` method manually sets and unsets the `http.Client`'s Timeout:

    ```go
    // Current approach in Gofile client
    func (c *Client) UploadFile(server string, filePath string, folderID string, callbackUpdate ProgressCallback) (*http.Response, error) {
        // ...
        c.httpClient.Timeout = 0 // Disable timeout for potentially long uploads
        req, err := http.NewRequest(http.MethodPost, u, pr)
        // ...
        response, err := c.httpClient.Do(req)
        // ...
        c.httpClient.Timeout = 1 * time.Minute // Reset timeout
        return response, nil
    }
    ```

This approach is a bit clunky. A better solution would be to use context.Context. For instance, context.WithDeadline could set a timeout that scales based on estimated upload speed and file size, allowing for more robust and flexible timeout management. Alternatively, I could use `http.NewRequestWithContext` instead, which enables attaching a context directly to the request without altering the global `http.Client`.

2. **Enhanced Concurrency**: I'd also like to leverage Go's powerful concurrency model more effectively. Currently, tasks are processed sequentially. I initially switched from channels to a map for task management, but the optimal solution likely involves a combination. My plan is to implement a global worker pool that can process a configurable number of tasks concurrently (e.g., 3 downloads at a time). This system would also need to track available resources, such as disk storage, to prevent issues like running out of space, especially if multiple large torrents are downloaded and then zipped. `sync.WaitGroup` could be employed to manage and synchronize these concurrent download goroutines.
    
## Takeaways and Reflections

For the frontend, I chose a combination of HTMX, Alpine.js, and Tailwind CSS. This was primarily driven by an enthusiasm to explore new technologies. In retrospect, while the backend carries most of the heavy lifting in this application, React (my usual go-to) might have been overkill.

My experience with this frontend stack was not entirely smooth, but HTMX's approach to enhancing HTML did provide valuable insights into DOM interactions and server-rendered partials. I later discovered other tools like DataStar and Templ UI (for Go templates) that might have eased frontend development further, albeit too late for this iteration.

Overall, building seedrlike has been a rewarding experience. It's actively deployed and I use it almost daily. The project has significantly boosted my understanding of Go and provided practical experience in tackling real-world application development challenges.
