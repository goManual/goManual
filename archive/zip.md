# Package zip

###Overview
Package zip provides support for reading and writing ZIP archives.<br>
读取和写入 ZIP归档<br>

See: http://www.pkware.com/documents/casestudies/APPNOTE.TXT<br>

This package does not support disk spanning.<br>
这个包不支持磁盘跨越

A note about ZIP64:<br>
To be backwards compatible the FileHeader has both 32 and 64 bit Size fields. The 64 bit fields will always contain the correct value and for normal archives both fields will be the same. For files requiring the ZIP64 format the 32 bit fields will be 0xffffffff and the 64 bit fields must be used instead.<br>
ZIP64注意:为了向后兼容,FileHeader里有32位和64位 Size 字段。64位通常包含的正确的值和正常的归档字段将是相同的。对于文件要求ZIP64格式, 32位字段将0xffffffff 并且必须使用64位字段。<br>

###Constants
```golang
const (
    Store   uint16 = 0
    Deflate uint16 = 8
)
Compression methods.
```
###Variables
```golang
var (
    ErrFormat    = errors.New("zip: not a valid zip file")
    ErrAlgorithm = errors.New("zip: unsupported compression algorithm")
    ErrChecksum  = errors.New("zip: checksum error")
)
```

##func RegisterCompressor
```golang
func RegisterCompressor(method uint16, comp Compressor)
```
RegisterCompressor registers custom compressors for a specified method ID. The common methods Store and Deflate are built in.<br>
RegisterCompressor 为指定的方法ID注册自定义压缩器。常见的方法Store和Deflate是内置<br>
<br><br>

###func RegisterDecompressor
```golang
func RegisterDecompressor(method uint16, d Decompressor)
```
RegisterDecompressor allows custom decompressors for a specified method ID.<br>
RegisterDecompressor 允许为指定的方法ID自定义解压器<br>
<br><br>


###type Compressor
```golang
type Compressor func(io.Writer) (io.WriteCloser, error)
```
A Compressor returns a compressing writer, writing to the provided writer. On Close, any pending data should be flushed.<br>
Compressor返回一个压缩的writer，writer提供写。Close时,任何挂起的数据都被刷新.<br>
<br><br>

###type Decompressor
```golang
type Decompressor func(io.Reader) io.ReadCloser
```
Decompressor is a function that wraps a Reader with a decompressing Reader. The decompressed ReadCloser is returned to callers who open files from within the archive. These callers are responsible for closing this reader when they're finished reading.<br>
Decompressor 是一个封装Reader和解压的Reader的函数。解压的ReadCloser返回给调用者从归档内打开的文件。这些调用者负责当他们结束读取时关闭这些reader.<br>
<br><br>


###type File
```golang
  type File struct {
        FileHeader
        // contains filtered or unexported fields
  }
```


###func (*File) DataOffset
```golang
func (f *File) DataOffset() (offset int64, err error)
```
DataOffset returns the offset of the file's possibly-compressed data, relative to the beginning of the zip file.<br>
Most callers should instead use Open, which transparently decompresses data and verifies checksums.<br>
DataOffset返回文件可能压缩数据的偏移,相对ZIP文件的开头.<br>
大部分的调用者应该使用Open 代替它,其透明地解压缩数据和验证校验。<br>
<br><br>

###func (f *File) Open
```golang
func (f *File) Open() (rc io.ReadCloser, err error)
```
Open returns a ReadCloser that provides access to the File's contents. Multiple files may be read concurrently.<br>
Open 返回ReadCloser那可以用来访问File的内容. 多个文件可以同时读取.<br>
<br><br>

###type FileHeader
```golang
  type FileHeader struct {
        // Name is the name of the file. 文件名称
        // It must be a relative path: it must not start with a drive 必须是一个真实存在的文件，必须处于未打开的状态
        // letter (e.g. C:) or leading slash, and only forward slashes  （就是相当于linux和windows目录开头）
        // are allowed.
        Name string

        CreatorVersion     uint16
        ReaderVersion      uint16
        Flags              uint16
        Method             uint16
        ModifiedTime       uint16 // MS-DOS time dos的时间
        ModifiedDate       uint16 // MS-DOS date dos的数据
        CRC32              uint32
        CompressedSize     uint32 // deprecated; use CompressedSize64
        UncompressedSize   uint32 // deprecated; use UncompressedSize64
        CompressedSize64   uint64
        UncompressedSize64 uint64
        Extra              []byte
        ExternalAttrs      uint32 // Meaning depends on CreatorVersion
        Comment            string
  }
```
FileHeader describes a file within a zip file. See the zip spec for details.<br>
FileHeader描述zip的文件信息<br>
<br><br>

###func FileInfoHeader
```golang
func FileInfoHeader(fi os.FileInfo) (*FileHeader, error)
```
FileInfoHeader creates a partially-populated FileHeader from an os.FileInfo.<br>
FileInfoHeader 从一个os.FileInfo创建部分填充的FileHeader<br>
<br><br>

###func (h *FileHeader) FileInfo
```golang
func (h *FileHeader) FileInfo() os.FileInfo
```
FileInfo returns an os.FileInfo for the FileHeader.<br>
FileInfo 为FileHeader返回一个os.FileInfo.<br>
<br><br>

###func (h *FileHeader) ModTime
```golang
func (h *FileHeader) ModTime() time.Time
```
ModTime returns the modification time. The resolution is 2s.<br>
ModTime 返回修改时间. 间隔2S.<br>
<br><br>

###func (h *FileHeader) Mode
```golang
func (h *FileHeader) Mode() (mode os.FileMode)
```
Mode returns the permission and mode bits for the FileHeader.<br>
Mode返回FileHeader的权限和模式位<br>
<br><br>

###func (h *FileHeader) SetModTime
```golang
func (h *FileHeader) SetModTime(t time.Time)
```
SetModTime sets the ModifiedTime and ModifiedDate fields to the given time. The resolution is 2s.<br>
SetModTime 使用给定的时间设置ModifiedTime和ModifiedDate字段。<br>
<br><br>

###func (h *FileHeader) SetMode
```golang
func (h *FileHeader) SetMode(mode os.FileMode)
```
SetMode changes the permission and mode bits for the FileHeader.<br>
SetMode 修改FileHeader的权限和模式位<br>
<br><br>

###type ReadCloser
```golang
  type ReadCloser struct {
        Reader
        // contains filtered or unexported fields
  }
```

###func OpenReader
```golang
  func OpenReader(name string) (*ReadCloser, error)
```
OpenReader will open the Zip file specified by name and return a ReadCloser.<br>
OpenReader打开指定名称的zip文件并返回ReadCloser.<br>
<br><br>

###func (rc *ReadCloser) Close
```golang
  func (rc *ReadCloser) Close() error
```
Close closes the Zip file, rendering it unusable for I/O.<br>
Close关闭zip文件. 使其无法使用I/O.<br>
<br><br>

###type Reader
```golang
  type Reader struct {
        File    []*File
        Comment string
        // contains filtered or unexported fields
  }
```
###▾ Example

Code:
```golang
//例子
// Open a zip archive for reading.
r, err := zip.OpenReader("testdata/readme.zip")
if err != nil {
    log.Fatal(err)
}
defer r.Close()

// Iterate through the files in the archive,
// printing some of their contents.
for _, f := range r.File {
    fmt.Printf("Contents of %s:\n", f.Name)
    rc, err := f.Open()
    if err != nil {
        log.Fatal(err)
    }
    _, err = io.CopyN(os.Stdout, rc, 68)
    if err != nil {
        log.Fatal(err)
    }
    rc.Close()
    fmt.Println()
}
```
<br><br>
Output:
```golang
Contents of README:
This is the source code repository for the Go programming language.
```
<br><br>

###func NewReader
```golang
func NewReader(r io.ReaderAt, size int64) (*Reader, error)
```
NewReader returns a new Reader reading from r, which is assumed to have the given size in bytes.<br>
NewReader 从r返回一个新Reader, 这假定其具有以字节为单位给定的尺寸。<br>
<br><br>

###type Writer
```golang
  type Writer struct {
    // contains filtered or unexported fields
  }
  Writer implements a zip file writer.
```
<br><br>

###▹ Example
```golang
// Create a buffer to write our archive to.
buf := new(bytes.Buffer)

// Create a new zip archive.
w := zip.NewWriter(buf)

// Add some files to the archive.
var files = []struct {
    Name, Body string
}{
    {"readme.txt", "This archive contains some text files."},
    {"gopher.txt", "Gopher names:\nGeorge\nGeoffrey\nGonzo"},
    {"todo.txt", "Get animal handling licence.\nWrite more examples."},
}
for _, file := range files {
    f, err := w.Create(file.Name)
    if err != nil {
        log.Fatal(err)
    }
    _, err = f.Write([]byte(file.Body))
    if err != nil {
        log.Fatal(err)
    }
}

// Make sure to check the error on Close.
err := w.Close()
if err != nil {
    log.Fatal(err)
}
```
<br><br>
###func NewWriter
```golang
func NewWriter(w io.Writer) *Writer
```
NewWriter returns a new Writer writing a zip file to w.<br>
NewWriter 返回一个新的Writer,写一个zip文件到w.<br>
<br><br>

###func (*Writer) Close
```golang
func (w *Writer) Close() error
```
Close finishes writing the zip file by writing the central directory. It does not (and can not) close the underlying writer.<br>
Close 结束写入ZIP文件。它不会（也不能）关闭底层writer<br>
<br><br>

###func (w *Writer) Create
```golang
func (w *Writer) Create(name string) (io.Writer, error)
```
Create adds a file to the zip file using the provided name. It returns a Writer to which the file contents should be written. The name must be a relative path: it must not start with a drive letter (e.g. C:) or leading slash, and only forward slashes are allowed. The file's contents must be written to the io.Writer before the next call to Create, CreateHeader, or Close.<br>
Create 使用提供的name添加一个文件到ZIP文件。文件内容可写入的情况下返回Writer。name必须是相对路径：不能以字母(e.g. C:) 或者前斜杠开头，只能用正斜杠。文件内容必须在下一次调用Create、CreateHeader 或 Close 前写入io.Writer.<br>
<br><br>

###func (w *Writer) CreateHeader
```golang
func (w *Writer) CreateHeader(fh *FileHeader) (io.Writer, error)
```
CreateHeader adds a file to the zip file using the provided FileHeader for the file metadata. It returns a Writer to which the file contents should be written. The file's contents must be written to the io.Writer before the next call to Create, CreateHeader, or Close.<br>
CreateHeader使用FileHeader的文件元数据 添加一个文件到ZIP文件. 文件可写入的情况下返回Writer. 文件内容必须在下一次调用Create、CreateHeader 或 Close 前写入io.Writer.<br>
<br><br>












