## Stream  vs Channel

#### Stream 

* 단방향통신만 가능
* 하나의 스트림으로 입력과 출력을 동시에 처리할 수 없다. 
* 입출력을 위해서는 input stream과 output stream 
* File, ByteArray, Data, Buffered 등의 스트림으로 확장할 수 있다.


##### Output Stream에 정의된 write method


` public abstract void write(int b) throws IOException;`


      Writes the specified byte to this output stream. The general
      contract for <code>write</code> is that one byte is written
      to the output stream. The byte to be written is the eight
      low-order bits of the argument <code>b</code>. The 24
      high-order bits(최상위 비트) of <code> b </code> are ignored.
      <p>  Subclasses of <code>OutputStream</code> must provide an
      implementation for this method.
     
      @param      b   the <code>byte</code>.
      @exception  IOException  if an I/O error occurs. In particular,
                  an <code>IOException</code> may be thrown if the
                  output stream has been closed.

	출력 스트림에게 특정 byte (b) 만큼 write 한다. 
 




` public void write(byte b[]) throws IOException {
       write(b, 0, b.length);
    }
`
   	
      Writes <code>b.length</code> bytes from the specified byte array
      to this output stream. The general contract for <code>write(b)</code>
      is that it should have exactly the same effect as the call
      <code>write(b, 0, b.length)</code>.
     
      @param      b   the data.
      @exception  IOException  if an I/O error occurs.
      @see        java.io.OutputStream#write(byte[], int, int)
     
    파라미터는 byte 배열. 내부적으로 write 함수가 있다. 
  
   
   
   
   
<pre><code> public void write(byte b[], int off, int len) throws IOException {
        if (b == null) {
            throw new NullPointerException();
        } else if ((off < 0) || (off > b.length) || (len < 0) ||
                   ((off + len) > b.length) || ((off + len) < 0)) {
            throw new IndexOutOfBoundsException();
        } else if (len == 0) {
            return;
        }
        for (int i = 0 ; i < len ; i++) {
            write(b[off + i]);
        }
    }</pre></code>
   
      Writes <code>len</code> bytes from the specified byte array
      starting at offset <code>off</code> to this output stream.
      The general contract for <code>write(b, off, len)</code> is that
      some of the bytes in the array <code>b</code> are written to the
      output stream in order; element <code>b[off]</code> is the first
      byte written and <code>b[off+len-1]</code> is the last byte written
      by this operation.
    
      The <code>write</code> method of <code>OutputStream</code> calls
      the write method of one argument on each of the bytes to be
      written out. Subclasses are encouraged to override this method and
      provide a more efficient implementation.
    
      If <code>b</code> is <code>null</code>, a
      <code>NullPointerException</code> is thrown.
    
      If <code>off</code> is negative, or <code>len</code> is negative, or
      <code>off+len</code> is greater than the length of the array
      <code>b</code>, then an <tt>IndexOutOfBoundsException</tt> is thrown.
     
      @param      b     the data.
      @param      off   the start offset in the data.
      @param      len   the number of bytes to write.
      @exception  IOException  if an I/O error occurs. In particular,
                  an <code>IOException</code> is thrown if the output
                  stream is closed.
    


### Example

<pre><code> public class ouputStream {

    private static final String OUTPUT_FILE = "C:\\Users\\outputex.txt";
    public static void main(String[] args) {

        String content = "Hello what the hell o my god";

        byte[] bytes = content.getBytes();

        try (OutputStream out = new FileOutputStream(OUTPUT_FILE)) {
        try (OutputStream out = new BufferedOutputStream(new FileOutputStream(OUTPUT_FILE),1024)) {

            // write a byte sequence
           out.write(bytes);    -- 바이트 배열만큼 write 하는 함수

            // write a single byte
            out.write(bytes[0]); -- 첫번째 byte 하나만 출력되는 함수

            // write sub sequence of the byte array
            out.write(bytes,4,10); - 4byte씩 끊어서 write 하는 함수

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
</pre><code>



* InputStream도 동일하다. 

#### 추가 진행할 것들?
 - Channel 클래스 
 - stream과 channel 성능 비교?
 - 뭐하지