# Package tar

Package tar implements access to tar archives. It aims to cover most of the variations, including those produced by GNU and BSD tars.<br>
实现访问tar归档文件。它的目的是覆盖大部分变种 ,包含GNU和BSD归档文件。

引用:<br>
[http://www.freebsd.org/cgi/man.cgi?query=tar&sektion=5](http://www.freebsd.org/cgi/man.cgi?query=tar&sektion=5)<br>
[http://www.gnu.org/software/tar/manual/html_node/Standard.html](http://www.gnu.org/software/tar/manual/html_node/Standard.html)<br>
[http://pubs.opengroup.org/onlinepubs/9699919799/utilities/pax.html](http://pubs.opengroup.org/onlinepubs/9699919799/utilities/pax.html)<br>


Example:
```golang
  // Create a buffer to write our archive to.
  buf := new(bytes.Buffer)

  // Create a new tar archive.
  tw := tar.NewWriter(buf)

  // Add some files to the archive.
  var files = []struct {
      Name, Body string
  }{
      {"readme.txt", "This archive contains some text files."},
      {"gopher.txt", "Gopher names:\nGeorge\nGeoffrey\nGonzo"},
      {"todo.txt", "Get animal handling licence."},
  }
  for _, file := range files {
      hdr := &tar.Header{
          Name: file.Name,
          Size: int64(len(file.Body)),
      }
      if err := tw.WriteHeader(hdr); err != nil {
          log.Fatalln(err)
      }
      if _, err := tw.Write([]byte(file.Body)); err != nil {
          log.Fatalln(err)
      }
  }
  // Make sure to check the error on Close.
  if err := tw.Close(); err != nil {
      log.Fatalln(err)
  }

  // Open the tar archive for reading.
  r := bytes.NewReader(buf.Bytes())
  tr := tar.NewReader(r)

  // Iterate through the files in the archive.
  for {
      hdr, err := tr.Next()
      if err == io.EOF {
          // end of tar archive
          break
      }
      if err != nil {
          log.Fatalln(err)
      }
      fmt.Printf("Contents of %s:\n", hdr.Name)
      if _, err := io.Copy(os.Stdout, tr); err != nil {
          log.Fatalln(err)
      }
      fmt.Println()
  }
```
<br><br>

###Constants
```golang
const (

    // Types
    TypeReg           = '0'    // regular file
    TypeRegA          = '\x00' // regular file
    TypeLink          = '1'    // hard link
    TypeSymlink       = '2'    // symbolic link
    TypeChar          = '3'    // character device node
    TypeBlock         = '4'    // block device node
    TypeDir           = '5'    // directory
    TypeFifo          = '6'    // fifo node
    TypeCont          = '7'    // reserved
    TypeXHeader       = 'x'    // extended header
    TypeXGlobalHeader = 'g'    // global extended header
    TypeGNULongName   = 'L'    // Next file has a long name
    TypeGNULongLink   = 'K'    // Next file symlinks to a file w/ a long name
)
```
<br><br>

###Variables
```golang
var (
    ErrWriteTooLong    = errors.New("archive/tar: write too long")
    ErrFieldTooLong    = errors.New("archive/tar: header field too long")
    ErrWriteAfterClose = errors.New("archive/tar: write after close")
)
```
```golang
var (
    ErrHeader = errors.New("archive/tar: invalid tar header")
)
```
<br><br>

###type Header
```golang
  type Header struct {
          Name       string    // name of header file entry 头文件名
          Mode       int64     // permission and mode bits 权限和模式位
          Uid        int       // user id of owner linux下的Uid
          Gid        int       // group id of owner 用户组
          Size       int64     // length in bytes 字节长度
          ModTime    time.Time // modified time 修改时间
          Typeflag   byte      // type of header entry 头文件类型
          Linkname   string    // target name of link 链接地址
          Uname      string    // user name of owner 用户名
          Gname      string    // group name of owner 用户组名
          Devmajor   int64     // major number of character or block device 大数量的字符或者块设备
          Devminor   int64     // minor number of character or block device 小数量的字符或者块设备
          AccessTime time.Time // access time 访问时间
          ChangeTime time.Time // status change time 状态更改时间
  }
```
A Header represents a single header in a tar archive. Some fields may not be populated.<br>
Header表示一个tar文档的头信息。 某些字段可以不用填充.
<br><br>

###func FileInfoHeader
```golang
func FileInfoHeader(fi os.FileInfo, link string) (*Header, error)
```
FileInfoHeader creates a partially-populated Header from fi. If fi describes a symlink, FileInfoHeader records link as the link target. If fi describes a directory, a slash is appended to the name. Because os.FileInfo's Name method returns only the base name of the file it describes, it may be necessary to modify the Name field of the returned header to provide the full path name of the file.<br>
FileInfoHeader 从fi创建部分填充的Header.如果fi是一个符号链接，FileInfoHeader 记录链接的链接目标，如果fi是一个目录，目录名加斜杠。因为os.FileInfo的 Name方法 返回它描述的文件的基本名称，要修改返回头的Name字段必须使用完整的路径名<br>
<br><br>

###func (h *Header) FileInfo
```golang
func (h *Header) FileInfo() os.FileInfo
```
FileInfo returns an os.FileInfo for the Header.<br>
FileInfo 返回一个os.FileInfo
<br><br>

###type Reader
```golang
  type Reader struct {
          // contains filtered or unexported fields 包涵过滤或取消导出字段
  }
```
A Reader provides sequential access to the contents of a tar archive. A tar archive consists of a sequence of files. The Next method advances to the next file in the archive (including the first), and then it can be treated as an io.Reader to access the file's data.<br>
Reader 顺序访问tar归档包的目录。一个tar归档由一系列文件组成。Next方法访问归档文件中的下一个文件（包含第一个文件）然后它可以像io.Reader一样访问文件的数据
<br><br>

###func NewReader
```golang
func NewReader(r io.Reader) *Reader
```
NewReader creates a new Reader reading from r.<br>
NewReader从r中创建新的Reader对象
<br><br>

###func (tr *Reader) Next
```golang
func (tr *Reader) Next() (*Header, error)
```
Next advances to the next entry in the tar archive<br>
Next 进入的tar归档下一个条目
<br><br>

###func (tr *Reader) Read
```golang
func (tr *Reader) Read(b []byte) (n int, err error)
```
Read reads from the current entry in the tar archive. It returns 0, io.EOF when it reaches the end of that entry, until Next is called to advance to the next entry.<br>
Read 从tar归档中读取当前的条目。 当它到该条目末尾时返回0,io.EOF ,直到调用Next前进到下一个条目.
<br><br>

###type Writer
```golang
  type Writer struct {
        // contains filtered or unexported fields
  }
```
A Writer provides sequential writing of a tar archive in POSIX.1 format. A tar archive consists of a sequence of files. Call WriteHeader to begin a new file, and then call Write to supply that file's data, writing at most hdr.Size bytes in total.<br>
Writer 提供用POSIX.1格式顺序写入归档。一个归档文件，由一系列文件组成。调用WriteHeader开始一个新文件，然后调用Write 提供该文件的数据，最多写入hdr.Size字节。
<br><br>

###func NewWriter
```golang
func NewWriter(w io.Writer) *Writer
```
NewWriter creates a new Writer writing to w.<br>
NewWriter创建一个新的Writer写入w。
<br><br>

###func (tw *Writer) Close
```golang
func (tw *Writer) Close() error
```
Close closes the tar archive, flushing any unwritten data to the underlying writer.<br>
Close 关闭归档，刷新任何未写入的数据到底层writer
<br><br>

###func (tw *Writer) Flush()
```golang
func (tw *Writer) Flush() error
```
Flush finishes writing the current file (optional).<br>
Flush 结束写入当前文件(可选).
<br><br>

###func (tw *Writer) Write
```golang
func (tw *Writer) Write(b []byte) (n int, err error)
```
Write writes to the current entry in the tar archive. Write returns the error ErrWriteTooLong if more than hdr.Size bytes are written after WriteHeader.<br>
Write 写入在tar归档中的当前条目. 如果在WriteHeader之后写入的字节超过hdr.Size，Write返回ErrWriteTooLong 错误.
<br><br>


###func (tw *Writer) WriteHeader
```golang
func (tw *Writer) WriteHeader(hdr *Header) error
```
WriteHeader writes hdr and prepares to accept the file's contents. WriteHeader calls Flush if it is not the first header. Calling after a Close will return ErrWriteAfterClose.<br>
WriteHeader写hdr并准备接收文件的内容.如果不是第一个header, WriteHeader调用Flush. 在Close 之后调用该方法将返回ErrWriteAfterClose
<br><br>
