# input & output

## 1. 입출력
- `I/O` (Input/Output)
- 컴퓨터 내부 or 외부의 장치와 프로그램간의 데이터를 주고받는 것

### 1.1 스트림
- 데이터를 운반하는데 사용되는 `연결통로` (단방향 통신)
- 입출력을 동시에 수행하기위해선 `입력스트림`, `출력스트림` 2개가 필요하다.
- `FIFO 구조`

### 1.2 바이트기반 스트림
- `InputStream`, `OutputStream`
- 스트림은 `byte`단위로 데이터를 전송한다.

- 입력스트림과 출력스트림의 종류

| 입출력 대상의 종류 | 입력 스트림 | 출력 스트림 |
|---|---|---|
| 파일 | FileInputStream | FileOutputStream |
| 메모리(byte배열) | ByteArrayInputStream | ByteArrayOutputStream |
| 프로세스(프로세스간의 통신) | PipedInputStream | PipedOutputStream |
| 오디오 장치 | AudioInputStream | AudioInputStream |
> InputStream or OutputStream의 자손들이며 각각읽고 쓰는데 필요한 추상메서드를 자신에 맞게 구현해 놓았다.

- InputStream과 OutputStream에 정의된 읽기와 쓰기를 수행하는 메서드

| InputStream | OutputStream |
|---|---|
| abstract int read() | abstract void write(int b) |
| int read(byte[] b) | void write(byte[] b) |
| int read(byte[] b, int off, int len) | void write(byte[] b, int off, int len) |
> read()의 반환타입이 byte가 아니라 int인 이유는 read()의 반환값의 범위가 0 ~ 255와 -1이기 때문이다.   
> 추상메서드인 read()와 write()를 이용해서 구현한 것이기 때문에 read()와 write()가 구현되어 있지 않으면 사용할 수 없다.
```java
public abstract class InputStream {
    ...
    // 입력스트림으로부터 1 byte를 읽어서 반환, 못읽으면 -1반환
    abstract int read();
    // 입력스트림으로부터 len개의 byte를 읽어서 byte배열 bdml off위치부터 저장
    int read(byte[] b, int off, int len) {
        ...
        for(int i = off; i < off + len; i++)
            // read()를 호출해서 데이터를 읽어서 배열을 채운다.
            b[i] = (byte)read();
    }
    // 입력스트림으로부터 byte배열 b의 크기만큼 데이터를 읽어서 배열 b에 저장한다.
    int read(byte[] b) {
        return read(b, 0, b.length);
    }
}
```
> 실제로는 추상클래스를 상속받아서 추상메서드를 구현한 클래스의 인스턴스에대해서 추상메서드가 호출될 것이기 때문에 </br> 추상메서드를 호출하는 코드를 작성해도 아무런 문제가 되지 않는다.

### 1.3 보조 스트림
- 스트림의 기능을 향상시키거나 새로운 기능을 추가
- 모든 보조스트림은 InputStream과 OutputStream의 자손들이며 입출력방식 또한 같다.
```java
// 기반 스트림 생성
FileInputStream fis = new FileInputStream("test.txt");
// 기반스트림을 이용해 보조스트림 생성
BufferdInputStream bis = new BufferedInputStream(fis);
// 보조스크리인 BufferedInputStream으로부터 데이터를 읽는다.
bis.read(); // 실제 입력기능은 보조스트림에 연결된 기반 스트림이 수행하고 보조스트림은 버퍼만 제공
```

- 보조스트림의 종류

| 입력 | 출력 | 설명 |
|---|---|---|
| FilterInputStream | FilterOutputStream | `필터`를 이용한 입출력 처리 |
| BufferedInputStreram | BufferedOutputStreram | `버퍼`를 이용한 입출력 성능향상 |
| DataInputStream | DataOutputStream | int, float와 같은 `기본형 단위`로 데이터를 처리하는 기능 | 
| SequenceInputStream | | 두 개의 스트림을 `하나로 연결` |
| LineNumberInputStream | | 읽어온 데이터를 라인 번호를 카운트 (jdk1.1부터 `LineNumberReader`로 대체) |
| ObjectInputStream | ObjectOutputStream | 데이터를 객체단위로 읽고 쓰는데 사용, 주로 파일을 이용하며 객체 `직렬화`와 관련 |
| | PrintStream | `버퍼`를 이용하며, 추가적인 `print관련` 기능(print, printf, pruntln메서드) |
| PushbackInputStream | | `버퍼`를 이용해서 읽어온 데이터를 다시 `되돌리는 기능`(unread, push back to buffer) |

### 1.4 문자기반 스트림
- `Reader`, `Writer`
- char형이 1byte가 아니라 2byte이기 때문에 바이트기반의 스트림으로는 2byte인 문자를 처리하는데 어려움이있다.
- 문자데이터를 입출력할 때는 바이트기반 스트림 대신 `문자기반 스트림`을 사용
- 바이트기반 스트림 vs 문자기반 스트림

| 바이트기반 스트림 | 문자기반 스트림 |
|---|---|
| FileInputStream </br> FileOutputStream | FileReader </br> FileWriter |
| ByteArrayInputStream </br> ByteArrayOutputStream | CharArrayReader </br> CharArrayWriter |
| PipedInputStream </br> PipedOutputStream | PipedReader </br> PipedWriter |
| StringBufferInputStream(deprecated) </br> StringBufferOutputStream(deprecated) | String Reader </br> StringWriter |

- 문자기반 스트림의 읽고 쓰는 메서드
: byte배열 대신 `char배열`을 사용한다는 것과 `추상메서드`가 달라졌다.

| InputStream/OutputStream | Reader/Writer |
|---|---|
| abstract int read() | int read() |
| int read(byte[] b) | int read(char[] cbuf) |
| int read(byte[], int off, int len) | abstract int read(char[] cbuf, int off, int len) |
| abstract void write(int b) | void write(int c) |
| void write(byte[] b) | void write(char[] cbuf) |
| void write(byte[] b, int off, int len) | abstract void write(char[] cbuf, int off, int len) |
| | void write(String str) |
| | void write(String str, int off, int len) |

- 문자기반 보조스트림
: InputStream/OutputStream -> Reader/Writer

## 2. 바이트기반 스트림

### 2.1 InputStream & OutputStream
- 스트림을 사용해서 모든 작업을 마치고 나면 `close()를 호출`해서 반드시 닫아주어야한다.
- ByteArrayInputStream과 같이 메모리를 사용하는 스트림과 System.in, System.out과 같은 표준 입출력의 스트림은 닫지 않아도 된다.

- InputStream의 메서드

| 메서드 | 설명 |
|---|---|
| int available() | 스트림으로부터 읽어 올 수 있는 데이터의 크기 반환 |
| void close() | 스트림을 닫음으로써 사용하고 있던 자원을 반환 |
| void mark(int readlimit) | 현재위치를 표시해 놓으며 후에 reset()에 의해서 표시해 놓은 위치로 다시 돌아갈 수 있다. </br> readlimit은 되돌아갈 수 있는 byte의 수이다. |
| boolean markSupported() | mark()와 reset()을 지원하는지 알려준다. mark()와 reset()기능을 지원하는 것은 선택적이므로, </br> mark()와 reset()을 사용하기 전에 markSupported()를 호출해서 지원여부를 확인 |
| abstract int read() | 1byte를 읽어 온다(0 ~ 255), 더 이상 읽어 올 데이터가 없으면 -1 반환 |
| int read(byte[] b) | 배열 b의 크기만큼 읽어서 배열을 채우고 읽어 온 데이터의 수를 반환,</br> 반환하는 값은 항상 배열의 크기보다 작거나 같다 |
| int read(byte[] b, int off, int len) | 최대 len개의 byte를 읽어서 배열 b의 지정된 위치(off)부터 저장, </br> 실제로 읽어 올 수 있는 데이터가 len개보다 적을 수 없다. |
| void reset() | 스트림에서의 위치를 마지막으로 mark()이 호출되었던 위치로 되돌린다. |
| long skip(long n) | 스트림에서 주어진 길이(n)만큼을 건너뛴다. |

- OutputStream의 메서드

| 메서드 | 설명 |
|---|---|
| void close() | 입력소스를 닫음으로써 사용하고 있던 자원을 반환 |
| void flush() | 스트림의 버퍼에 있는 모든 내용을 출력소스에 쓴다. |
| abstract void write(int b) | 주어진 값을 출력소스에 쓴다. |
| void write(byte[] b) | 주어진 배열 b에 저장된 모든 내용을 출력소스에 쓴다. |
| void write(byte[] b, int off, int len) | 주어진 배열 b에 저장된 내용 중에서 off번째부터 len개 만큼만을 읽어서 출력소스에 쓴다. |
> Flush()는 버퍼가 있는 출력스트림의 경우에만 의미가 있으며 OutputStream에 정의된 flush()는 아무런 일도 하지 않는다.

### 2.2 ByteArrayInputStream & ByteArrayOutputStream
- 바이트 배열에 데이터를 입출력하는데 사용되는 스트림 (자주 사용하지는 않는다.)

```java
import java.io.*;
import java.util.Arrays;

public class IoEx4 {

	public static void main(String[] args) {
        // byte 배열을 생성
		byte[] inSrc = {0,1,2,3,4,5,6,7,8,9};
		byte[] outSrc = null;
		byte[] temp = new byte[4];
		// byte기반 스트림 선언
		ByteArrayInputStream input = null;
		ByteArrayOutputStream output = null;
		// inSrc를 받는 byte기반의 입력 스트림 생성
		input = new ByteArrayInputStream(inSrc);
        // byte기반의 출력 스트림 생성
		output = new ByteArrayOutputStream();
		// read(),write()가 IOException 예외를 발생시킬 수 있으므로 예외 처리
		try {
            // blocking이 없다면
			while(input.available() > 0) {
                // 입력 스트림을 통해 얻어온 inSrc배열을 4 byte 배열인 temp에 읽어오고 읽어온 데이터의 개수를 반환
				int len = input.read(temp);
                // temp에 읽어온 만큼만 출력 스트림을 통해 write
				output.write(temp, 0, len);
			}
		} catch(IOException e) {}
		
		outSrc = output.toByteArray();
		
		System.out.println("Input Source : " + Arrays.toString(inSrc));
		System.out.println("temp Source : " + Arrays.toString(temp));
		System.out.println("Output Source : " + Arrays.toString(outSrc));
	}	
}
```
> blocking : 데이터를 읽어 올 때 데이터를 기다리기 위해 멈춰있는 것

### 2.3 FileInputStream & FileOutputStream
- 파일에 입출력을 하기 위한 스트림

- 생성자

| 생성자 | 설명 |
|---|---|
| FileInputStream(String name) | 지정된 파일이름을 가진 실제 파일과 연결된 FileInputStream을 생성 |
| FileInputStream(File file) | 파일의 이름이 String이 아닌 File인스턴스로 지정해주어야 하는 점을 제외하고 </br> FileInputStream(String name)와 같다. |
| FileOutputStream(FileDescriptor fdObj) | 파일 디스크립터로 FileInputStream을 생성 |
| FileOutputStream(String name) | 지정된 파일이름을 가진 실제 파일과의 연결된 FileOutputStream을 생성 |
| FileOutputStream(String name, boolean append) | 지정된 파일이름을 가진 실제 파일과의 연결된 FileOutputStream을 생성, </br> append를 true -> 덧붙이기, false -> 덮어쓰기 |
| FileOutputStream(File file) | 파일의 이름을 String이 아닌 File인스턴스로 지정해주어야 하는 점을 제외하고 </br> FileOutputStream(String name)과 같다. |
| FileOutputStream(File file, boolean append) | 파일의 이름을 String이 아닌 File인스턴스로 지정해주어야 하는 점을 제외하고 </br> FileOutputStream(String name, boolean append)과 같다. |
| FileOutputStream(FileDescriptor fdObj) | 파일 디스크립터로 FileOutputStream을 생성 |

```java
// FileCopy.java의 내용을 read()로 읽어서, write(int b)로 FileCopy.bak에 출력
import java.io.*;

class FileCopy {
    public static void main(String[] args) {
        try {
            // FileCopy.java 소스파일을 받는 입력 스트림을 생성
            FileInputStream fis = new FileInputStream(args[0]);
            // FileCopy.bak으로 출력하는 출력 스트림을 생성
            FileOutputStream fos = new FileOutputStream(args[1]);

            int data = 0;
            // 입력스트림을 통해 데이터를 read 성공한 경우
            while((data = fis.read()) != -1) {
                // 출력 스트림을 통해 데이터를 write
                fos.write(data);
            }
            // 입출력 스트림을 모두 닫는다.
            fis.close();
            fos.close();
        } catch (IOException e) {
            e.printStactTrace();
        }
    }
}

// 실행 코드 : java FileCopy FileCopy.java FileCopy.bak    (arg[0] : FileCopy.java, arg[1] : FileCopy.bak)
//           type FileCopy.bak
/* 그대로 복사 */
```
> 텍스트 파일을 다루는 경우에는 FileReader/FileWriter를 사용하는 것이 더 좋다.

## 3. 바이트기반의 보조스트림

### 3.1 FilterInputStream & FilterOutputStream
- `InputStream/OutputStream의 자손`이며 `모든 보조스트림의 조상`
- FilterInputStream/FilterOutputStream의 모든 메서드는 단순히 기반스트림의 메서드를 그대로 호출할 뿐 자체로는 아무런 기능을 하지않으므로 </br> 
`상속`을통해 메서드들을 원하는 작업을 수행하도록 `오버리이딩`한다.

- 생성자
```java
protected FilterInputSteram(InputStream in) // 인스턴스 생성 x (protected), 상속을 통해 오버리이딩 o 
public FilterOutputStream(OutputStream out)
```
- FilterInputStream/FilterOutputStream을 상속받아 기반스트림에 보조기능을 추가한 `보조스트림 클래스`
    + `FilterInputStream의 자손` : BufferedInputStream, DataInputSteram, PushbackInputStream ...
    + `FilterOutputStream의 자손` : BufferedOutputStream, DataOutputStream, PrintStream ...

### 3.2 BufferedInputStream & BufferedOutputStream
- 스트림의 입출력 효율을 높이기 위해 버퍼를 사용하는 보조스트림 (대부분의 입출력 작업에 사용)
- 외부소스로 부터 읽는 것보다 내부의 버퍼로 부터 읽는 것이 훨씬 빠르기 때문에 작업효율이 높아진다.
- 입력소스가 파일인 경우 보통 8192 byte(8k)로 한다.

- BufferedInputStream의 생성자 

| 생성자 | 설명 |
|---|---|
| BufferedInputStream(InputStream in, int size) | 주어진 InputStream인스턴스를 입력소스로 하며 지정된 크기(byte)의 버퍼를 갖는 BufferedInputStream인스턴스를 생성 |
| BufferedInputStream(InputStream in) | 주어진 InputStream인스턴스를 입력소스로 하며 기본적으로 8192 byte(8k)크기의 버퍼를 갖게된다. |

- BufferedOutputStream의 생성자와 메서드

| 생성자/메서드 | 설명 |
|---|---|
| BufferedOutputStream(OutputStream out, int size) | 주어진 OutputStream인스턴스를 출력소스로하며 지정된 크기(byte)의 버퍼를 갖는 BufferedOutputStream인스턴스를 생성 |
| BufferedOutputStream(OutputStream out) | 주어진 OutputStream인스턴스를 출력소스로하며 기본적으로 8192 byte 크기의 버퍼를 갖는다. |
| flush() | 버퍼의 모든 내용을 출력소스에 출력한 다음 버퍼를 비운다. |
| close() | flush()를 호출해서 버퍼의 모든 내용을 출력소스에 출력하고 BufferedOutputStream인스턴스가 </br> 사용하던 모든 자원을 반환 |

- `read()`
    + 입력소스로부터 버퍼의 크기만큼의 데이터를 읽어다 자신의 내부 버퍼에 저장
    + 프로그램에서 버퍼에 저장된 데이터를 읽는다.
- `write()`
    + 프로그램에서 write메서드를 이용한 출력이 버퍼에 저장
    + 버퍼가 가득차면 버퍼의 모든 내용을 출력소스에 출력
    + 버퍼를 비우고 다시 프로그램으로부터의 출력을 저장할 준비
> `주의` : 마지막 출력부분이 버퍼에 남아있는채로 프로그램이 종료될 수 있으므로 close()나 flush()를 호출

```java
import java.io.*;

class BufferedOutputStreamEx1 {
    public static void main(String args[]) {
        try {
            // 기반 출력 스트림 생성
            FileOutputStream fos = new FileOutputStream("123.txt");
            // 크기가 5인 출력 버퍼 생성
            BufferedOutputStream bos = new BufferedOutputStream(fos, 5);
            // 파일 123.text에 1부터 9까지 출력
            for(int i = '1'; i <= '9'; i++) {
                bos.write(i);
            }
            // 버퍼에 남아있는 내용 출력 후 스트림 닫기
            bos.close()
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// BufferedOutputStream의 close()는 기반 스트림의 close()를 호출하기 때문에 따로 호출해서 닫아줄 필요없다.
```

### 3.3 DataInputStream & DataOutputStream
- DataInputStream/DataOutputStream은 DataInput/DataOutput인터페이스를 구현하였기때문에 </br> 데이터를 읽고 쓰는데 있어서 byte단위가 아닌 `8가지 기본 자료형`의 단위로 읽고 쓸 수 있다.
- DataOutputStream이 출력하는 형식은 각 기본 자료형 값을 `16진수`로 표현하여 `저장`
- 데이터를 변환할 필요도없고, 자리수를 세어서 따지지 않아도 되므로 편리하고 빠르게 데이터를 저장하고 읽을 수 있다.

- DataInputStream의 생성자와 메서드

| 생성자/메서드 | 설명 |
|---|---|
| DataInputStream(InputStream in) | 주어진 InputStream인스턴스를 기반스트림으로 하는 DataInputStream인스턴스를 생성 |
| boolean readBoolean() </br> byte readByte() </br> char readChar() </br> short readShort() </br> int readInt() </br> long readLong() </br> float readFloat() </br> double readDouble() </br> int readUnsignedByte() </br> int readUnsignedShort() | 각 타입에 맞게 값을 읽어 온다. </br> 더 이상 읽을 값이 없으면 `EOFException`을 발생 |
| void readFully(byte[] b) </br> void readFully(byte[] b, int off, int len) | 입력 스트림에서 지정된 배열의 크기만큼 or 지정된 위치에서 len만큼 데이터를 읽어온다. </br> 파일의 끝에 도달하면 `EOFException`이 발생, I/O에러가 발생하면 `IOException`이 발생 |
| String readUTF() | UTF-8형식으로 쓰여진 문자를 읽는다. </br> 더 이상 읽을 값이 없으면 `EOFException`이 발생 |
| static String readUTF(DataInput in) | 입력스트림(in)에서 UTF-8형식의 유니코드를 읽어온다. |
| int skipBytes(int n) | 현재 읽고 있는 위치에서 지정된 숫자(n)만큼을 건너뛴다. |

- DataOutputStream의 생성자와 메서드

| 생성자/메서드 | 설명 |
|---|---|
| DataOutputStream(OutputStream out) | 주어진 OutputStream인스턴스를 기반스트림으로 하는 DataOutputStream인스턴스 생성 |
| void writeBoolean(boolean b) </br> void writeByte(int b) </br> void writeChar(int c) </br> void writeChars(String s) </br> void writeShort(int s) </br> void writeInt(int i) </br> void writeLong(long i) </br> void writeFloat(float f) </br> void writeDouble(double d) | 각 자료형에 알맞은 값들을 출력 |
| void writeUTF(String s) | UTF형식으로 문자를 출력 |
| void writeChars(String s) | 주어진 문자열을 출력 </br> writeChar(int c)를 여러 번 호출한 결과와 같다. |
| int size() | 지금가지 DataOutputStream에 쓰여진 byte의 수를 알려준다. |

```java
import java.io.*;

public class DataInputStreamEx2 {

	public static void main(String[] args) {
		int sum = 0;
		int score = 0;
		// 기반 스트림 선언
		FileInputStream fis = null;
		// 보조 스트림 선언
		DataInputStream dis = null;
		// readInt()은 예외를 발생할 수 있으므로 예외처리
		try {
			// score.dat파일로의 입력스트림 생성
			fis = new FileInputStream("score.dat");
			// 기반 스트림의 보조스트림 생성
			dis = new DataInputStream(fis);
			
			while(true) {
				// 무한루프를 이용해 score.dat의 내용을 읽어온다.
				score = dis.readInt();
				// 읽어온 내용을 출력
				System.out.println(score);
				sum += score;
			}
		} catch (EOFException e) {
			System.out.println("점수의 총합은 " + sum + "입니다.");
		} catch (IOException ie) {
			ie.printStackTrace();
		} finally {	// 작업도중에 예외가 발생해서 스트림을 닫지 못하고 try블럭을 빠져나갈 수 있기 때문에 finally블럭에서 스트림을 닫도록 처리
			try {
				// dis가 null일 때 close()호출할 경우 NullPointerException 발생
				if(dis != null)
					dis.close();
			} catch(IOException ie) {
				ie.printStackTrace();
			}
		}
	}
}
```
> 스트림을 사용한 직후에 바로 닫아서 자원을 반환하는 것이 좋다.

### 3.4 SequenceInputStream
- 여러 개의 입력스트림을 연속적으로 연결해서 하나의 스트림으로부터 데이터를 읽는 것과 같이 처리할 수 있도록 도와주는 보조 스트림
- 다른 보조 스트림들과는 달리 FilterInputStream의 자손이 아닌 `InputStream을 바로 상속받아서 구현`
- 생성자를 제외하면 나머지 작업은 다른 입력스트림과 다르지 않다.

- SequenceInputStream의 생성자와 메서드

| 생성자/메서드 | 설명 |
|---|---|
| SequenceInputStream(Enumeration e) | Enumeration에 저장된 순서대로 입력스트림을 하나의 스트림으로 연결 |
| SequenceInputStream(InputStream s1, InputStream s2) | 두 개의 입력스트림을 하나로 연결 |

```java
// 사용 예시
Vector files = new Vector();
files.add(new FileInputStream("file.001"));
files.add(new FileInputStream("file.002"));
SequenceInputStream int = new SequenceInputStream(files.elements());
```

### 3.5 PrintStream
- 데이터를 기반스트림에 다양한 형태로 출력할 수 있는 print, println, printf와 같은 메서드를 오버로딩하여 제공
- 데이터를 문자로 출력하는 것이기 때문에 문자기반 스트림의 역할을 수행
- 거의 같은 기능을 갖지만 가능한 PrintStream에 비해 다양한 언어의 문자를 처리하는데 적합한 `PrintWriter`를 사용하는 것이 좋다.

- PrintStream의 생성자와 메서드

| 생성자/메서드 | 설명 |
|---|---|
| PrintStream(File file) </br> PrintStream(File file, String csn) </br> PrintStream(OutputStream out) </br> PrintStream(OutputStream out, boolean autoFlush) </br> PrintStream(OutputStream out, boolean autoFlush, String encoding) </br> PrintStream(String fileName) </br> PrintStream(String fileName, String csn) | 지정된 출력스트림을 기반으로 하는 PrintStream인스턴스 생성, </br> autoFlush의 값을 true로 하면 println메서드가 호출되거나 </br> 개행문자가 출력될 때 자동으로 flush가된다. 기본값은 false |
| boolean checkError() | 스트림을 flush하고 에러가 발생했는지 알려준다. |
| void print(arg)/ void println(arg) </br> arg : boolean, char, char[], double, float, int, long, Object, String | 인자로 주어진 값을 출력소스에 문자로 출력 (줄바꿈차이) |
| void println() | 줄바꿈 문자 출력 |
| PrintStream printf(String format, Object... args) | 정형화된 출력을 가능하게 한다. |
| protected void setError() | 작업 중에 오류가 발생했음을 알린다. </br> setError()를 호출하고 checkError()를 호출하면 true반환 |

- `printf()`
    + jdk1.5부터 추가
    + C언어와 사용법 동일

## 4. 문자기반 스트림
- `문자데이터`를 다루는데 사용된다는 것을 제외하고 바이트기반 스트림과 사용방법은 거의 같다.

### 4.1 Reader & Writer
- 바이트기반 스트림의 조상이 InputStream/OutputStream인 것처럼 문자기반 스트림에서는 `Reader/Writer`가 같은 역할을 한다.
- `byte[]` -> `char[]`, `abstract` 변화

- Reader의 메서드

| 메서드 | 설명 |
|---|---|
| void close() | 스트림을 닫음으로써 사용하고 있던 자원을 반환 |
| void mark(int readlimit) | 현재위치를 표시해 놓으며 후에 reset()에 의해서 표시해 놓은 위치로 다시 돌아갈 수 있다. </br> readlimit은 되돌아갈 수 있는 byte의 수이다. |
| boolean markSupported() | mark()와 reset()을 지원하는지 알려준다. mark()와 reset()기능을 지원하는 것은 선택적이므로, </br> mark()와 reset()을 사용하기 전에 markSupported()를 호출해서 지원여부를 확인 |
| int read() | 입력소스로부터 하나의 문자를 읽어온다. char의 범위인 0 ~ 65535범위의 정수를 반환, </br> 더 이상 읽어 올 데이터가 없으면 -1 반환 |
| int read(char[] c) | 입력소스로부터 매개변수로 주어진 배열 c의 크기만큼 읽어서 </br> 배열을 채우고 읽어 온 데이터의 수 or -1 반환|
| abstract int read(char[] c, int off, int len) | 입력소스로부터 최대 len개의 문자를 읽어서 배열 c의 지정된 위치(off)부터 저장, </br> 읽어온 데이터의 개수 or -1 반환 |
| int read(CharBuffer target) | 입력소스로부터 읽어서 문자버퍼(target)에 저장 |
| boolean ready() | 입력소스로부터 데이터를 읽을 준비가 되어있는지 알려준다. |
| void reset() | 스트림에서의 위치를 마지막으로 mark()이 호출되었던 위치로 되돌린다. |
| long skip(long n) | 현재 위치에서 주어진 문자 수(n)만큼 건너뛴다. |

- Writer의 메서드

| 메서드 | 설명 |
|---|---|
| Writer append(char c) | 지정된 문자를 출력소스에 출력 |
| Writer append(CharSequence c) | 지정된 문자열(CharSequence)을 출력소스에 출력 |
| Writer append(CharSequence c, int start, int end) | 지정된 문자열의 일부를 출력소스에 출력 </br> (CharBuffer, String, StringBuffer가 CharSequence 구현)
| abstract void close() | 출력스트림을 닫음으로써 사용하고 있던 자원을 반환 |
| abstract void flush() | 스트림의 버퍼에 있는 모든 내용을 출력소스에 쓴다. |
| void write(int b) | 주어진 값을 출력소스에 쓴다. |
| void write(char[] c) | 주어진 배열 c에 저장된 모든 내용을 출력소스에 쓴다. |
| abstract void write(char[] c, int off, int len) | 주어진 배열 c에 저장된 내용 중에서 off번째부터 len개 만큼만을 읽어서 출력소스에 쓴다. |
| void write(String str) | 주어진 문자열(str)을 출력소스에 쓴다. |
| void write(String str, int off, int len) | 주어진 문자열의 일부(off부터 len까지)를 출력소스에 쓴다. |
> flush()는 버퍼가 있는 출력스트림의 경우에만 의미가 있다.

### 4.2 FileReader & FileWriter
- `파일`로부터 `텍스트데이터`를 읽고 쓰는데 사용
- FileInputStream/FileOutputStream과 사용방법이 같다.

### 4.3 PipedReader & PipedWriter
- `쓰레드` 간의 `데이터`를 주고받을 때 사용
- 다른 스트림과는 달리 `입력과 출력스트림을 하나의 스트림으로 연결`해서 데이터를 주고받는다.
- 스트림을 생성한 후 한쪽 쓰레드에서 `connect()`를 호출해서 입력스트림과 출력스트림을 연결
- 입출력을 마친 후 한쪽 스트림만 닫아도 나머지 스트림은 자동으로 닫힌다.

### 4.4 StringReader & StringWriter
- CharArrayReader/CharArrayWriter와 같이 입출력 대상이 `메모리`인 스트림
- StringWriter에 출력되는 데이터는 내부의 StringBuffer에 저장
- StringBuffer에 저장된 데이터를 얻을 수 있는 메서드
```java
StringBuffer getBuffer()    // StringWriter에 출력한 데이터가 저장된 StringBuffer를 반환
String toString()           // StringWriter에 출력된 문자열 반환 (StringBuffer에 저장된)
```

## 5. 문자기반의 보조스트림

### 5.1 BufferedReader & BufferedWriter
- `버퍼`를 이용해서 입출력의 효율을 높일 수 있도록 해주는 역할
```java
readLine()  // 데이터를 라인단위로 읽는다.
newLine()   // 줄바꿈
```

### 5.2 InputStreamReader & OutputStreamWriter
- 바이트기반 스트림을 문자기반 스트림으로 연결시켜주는 역할
- 바이트기반 스트림의 데이터를 지정된 인코딩의 문자데이터로 변환하는 작업을 수행
- InputStreamReader의 생성자와 메서드

| 생성자/메서드 | 설명 |
|---|---|
| InputStreamReader(InputStream in) | OS에서 사용하는 기본 인코딩의 문자로 변환하는 InputStreamReader를 생성 |
| InputStreamReader(InputStream in, String encoding) | 지정된 인코딩을 사용하는 InputStreamReader를 생성 |
| String getEncoding() | InputStreamReader의 인코딩을 알려준다. |

- OutputStreamWriter의 생성자와 메서드

| 생성자/메서드 | 설명 |
|---|---|
| OutputStreamWriter(OutputStream out) | OS에서 사용하는 기본 인코딩의 문자로 변환하는 OutputStreamWriter를 생성 |
| OutputStreamWriter(OutputStream out, String encoding) | 지정된 인코딩을 사용하는 OutputStreamWriter를 생성 | 
| String getEncoding() | OutputStreamWriter의 인코딩을 알려준다. |

## 6. 표준입출력과 File

### 6.1 표준입출력
- `콘솔`을 통한 데이터 입력과 콘솔로의 데이터 출력을 의미
- `System.in`, `System.out`, `System.err`
```java
System.in   // 콘솔로부터 데이터를 입력
System.out  // 콘솔로 데이터를 출려
System.err  // 콘솔로 데이터를 출력
```

### 6.2 표준입출력의 대상변경
- `콘솔이외`에 다른 입출력 대상으로 변경
- jdk1.5부터 `Scanner클래스`가 제공되면서 System.in으로부터 데이터를 입력받아 작업하는 것이 편리
- `setOut()`, `setErr()`, `setIn()`
```java
static void setOut(PrintStream out) // System.out의 출력을 지정한 PrintStream으로 변경
static void setErr(PrintStream err) // System.err의 출력을 지정한 PrintStream으로 변경
static void setIn(InputStream in)   // System.in의 입력을 지정한 InputStream으로 변경
```
- 커맨드라인에서 표준입출력 대상 바꾸는 방법
```java
// 지정된 파일에 내용이 있으면 내용 삭제
java 소스파일 > 지정한 파일
// 지정된 파일에 내용이 있으면 뒤에 덧붙인다.
java 소스파일 >> 지정한 파일
```

### 6.3 RandomAccessFile
- 파일의 `어느 위치`에나 읽기/쓰기가 가능하게하는 클래스
- InputStream/OutputStream을 상속받은 것이아닌 `DataInput/DataOutput인터페이스`를 구현했기 때문에 하나의 클래스로 파일에 대한 입력과 출력을 모두 가능
- `기본자료형` 단위로 데이터를 읽고 쓰기 가능
> 모든 입출력에 사용되는 클래스들은 입출력 시 다음 작업이 이루어질 위치를 저장하고 있는 포인터를 내부적으로 갖고 </br> 작업자가 포인터의 위치를 마음대로 변경할 수 없지만 RandomAccessFile은 가능하다.

- RandomAccessFile의 생성자와 메서드

| 생성자/메서드 | 설명 |
|---|---|
| RandomAccessFile(File file, String mode) </br> RandomAccessFile(String fileName, String mode) | 주어진 file에 읽기 또는 쓰기를 하기 위한 RandomAccessFile인스턴스를 생성 </br> mode : </br> "r" - 파일로부터 읽기만 수행할 때 </br> "rw" - 파일에 읽기와 쓰기(지정된 파일이 없으면 새로운 파일 생성) </br> "rws","rwd" - 기본적으로 "rw"와 같으며 출력내용이 파일에 지연 없이 바로 쓰이게한다. </br> (rws - 파일의 메타정보 포함, rwd - 파일의 내용만) |
| FileChannel getChannel() | 파일의 파일 채널을 반환 |
| FileDescriptor getFD() | 파일의 파일 디스크립터를 반환 |
| long getFilePointer() | 파일 포인터의 위치를 알려준다. |
| long length() | 파일의 크기를 알려준다.(byte) |
| void seek(long pos) | 파일 포인터의 위치를 변경, 위치 : 파일의 첫 부분부터 pos크기만큼 떨어진 곳 |
| void setLength(long newLength) | 파일의 크기를 지정된 길이로 변경 (byte) |
| int skipBytes(int n) | 지정된 수만큼의 byte를 건너뛴다. |

### 6.4 File
- 기본적이면서 가장 많이 사용되는 입출력 대상
- `파일`과 `디렉토리`를 다룰 수 있게하는 클래스
- File의 생성자와 경로와 관련된 메서드
- `절대경로` : 파일시스템의 루트(root)로부터 시작하는 파일의 전체 경로 (하나의 파일에 둘 이상의 절대경로 존재 가능)
- `정규경로` : 기호나 링크 등을 포함하지 않는 유일한 경로

| 생성자/메서드 | 설명 |
|---|---|
| File(String fileName) | 주어진 문자열을 이름으로 갖는 파일을 위한 File인스턴스를 생성, 디렉토리도 같은방법으로 생성, </br> fileName은 주로 path를 포함해서 지정해주지만, 파일 이름만 사용할 경우 실행되는 위치가 path로 간주 |
| File(String pathName, String fileName) </br> File(File pathName, String fileName) | 파일의 경로와 이름을 따로 분리해서 지정할 수 있도록 한 생성자 |
| File(URI uri) | 지정된 uri로 파일을 생성 |
| String getName() | 파일이름을 String으로 반환 |
| String getPath() | 파일의 경로를 String으로 반환 |
| String getAbsolutePath() </br> File getAbsoluteFile() | 파일의 절대경로를 String/File으로 반환 |
| String getParent() </br> File getParentFile() | 파일의 조상 디렉토리를 String/File로 반환 |
| String getCanonicalPath() </br> File getCanonicalFile() | 파일의 정규경로를 String/File로 반환 |

- 경로와 관련된 File의 멤버변수

| 멤버변수 | 설명 |
|---|---|
| static String pathSeparator | OS에서 사용하는 경로 구분자 (원도우 : ";", 유닉스 : ":") |
| static char pathSeparatorChar | OS에서 사용하는 경로 구분자 (원도우 : ';', 유닉스 : ':') |
| static String separator | OS에서 사용하는 이름 구분자 (윈도우 : "₩", 유닉스 : "/") |
| static char separatorChar | OS에서 사용하는 이름 구분자 (윈도우 : '₩', 유닉스 : '/') |

- File 인스턴스 생성하는 방법   
: File인스턴스를 생성하는 것이지 파일이나 디렉토리가 생성되는 것은 아니다.
```java
File f = new File("c:\\jdk1.8\\work\\ch15", "FileEx1.java");
File dir = new File("c:\\jdk1.8\\work\\ch15");
File f = new File(dir, "FileEx1.java");
```

- 파일 생성
```java
// 이미 존재하는 파일을 참조할 때
File f = new File("c:\\jdk1.8\\work\\ch15", "FileEx1.java");
// 기존에 없는 파일을 새로 생성할 때
File f = new File("c:\\jdk1.8\\work\\ch15", "FileEx1.java");
f.createNewFile();
```

- File의 메서드

| 메서드 | 설명 |
|---|---|
| boolean canRead() | 읽을 수 있는 파일인지 검사 |
| boolean canWrite() | 쓸 수 있는 파일인지 검사 |
| boolean canExecute() | 실행할 수 있는 파일인지 검사 |
| int compareTo(File pathname) | 주어진 파일 or 디렉토리 비교 (같으면 0, 다르면 1 or -1 반환) |
| boolean exists() | 파일이 존재하는지 검사 |
| boolean isAbsolute() | 파일 or 디렉토리가 절대경로명으로 지정되었는지 확인 |
| boolean isDirectory() | 디렉토리인지 확인 |
| boolean isFile() | 파일인지 확인 |
| boolean isHidden() | 파일의 속성이 '숨김'인지 확인 (파일이 존재하지 않으면 false) |
| boolean createNewFile() | 아무런 내용이 없는 새로운 파일 생성 (이미 존재할 경우 생성 x) |
| static File createTempFile(String prefix, String suffix) | 임시파일을 시스템의 임시 디렉토리에 생성 |
| static File createTempFile(String prefix, String suffix, File directory) | 임시파일을 시스템의 지정된 디렉토리에 생성 |
| boolean delete() | 파일을 삭제 |
| void deleteOnExit() | 응용 프로그램 종료시 파일을 삭제 |
| boolean equals(Object obj) | 주어진 객체(주로 File인스턴스)가 같은 파일인지 비교 |
| long lastModified() | 파일의 마지막으로 수정된 시간을 반환 |
| long length() | 파일의 크기 반환 |
| String[] list() | 디렉토리의 파일목록(디렉토리 포함)을 String배열로 반환 |
| String[] list(FilenameFilter filter) </br> File[] list(FilenameFilter filter) | FilenameFilter인스턴스에 구현된 조건에 맞는 파일을 String/File배열로 반환 |
| File[] listFiles() </br> File[] listFiles(FileFilter filter) </br> File[] listFiles(FilenameFilter f) | 디렉토리의 파일목록(디렉토리 포함)을 File배열로 반환 </br> (filter가 지정된 경우 filter의 조건과 일치하는 파일만 반환) |
| static File[] listRoots() </br> long getFreeSpace() </br> long getTotalSpace() </br> long getUsableSpace() | 컴퓨터의 파일시스템의 root의 목록(floppy, CD-ROM, HDD drive)을 반환 </br> (예 : A:₩, C:₩, D:₩) |
| boolean mkdir() </br> boolean mkdirs() | 파일에 지정된 경로로 디렉토리(폴더)를 생성 mkdirs는 필요하면 부모 디렉토리까지 생성 |
| boolean renameTo(File dest) | 지정된 파일(dest)로 이름을 변경 |
| boolean setExecutable(boolean executable) </br> boolean setExecutable(boolean executable, boolean ownerOnly) </br> boolean setreadable(boolean readable) </br> boolean setreadable(boolean readable, boolean ownerOnly) </br> boolean setReadOnly() </br> boolean setWritable(boolean writable) </br> boolean setWritable(boolean writable, boolean ownerOnly) | 파일의 속성을 변경, OwnerOnly가 true면 파일의 소유자만 해당 속성을 변경할 수 있다. |
| boolean setLastModified(long t) | 파일의 마지막으로 수정된 시간을 지정된 시간으로 변경 |
| Path topath() | 파일을 Path로 변환해서 반환 |
| URI toURI() | 파일을 URI로 변환해서 반환 |

## 7. 직렬화
- 객체를 데이터 스트림으로 만드는 것
    + 즉, 객체에 저장된 데이터를 스트림에 쓰기(write)위해 연속적인 데이터로 변환하는 것
- `역직렬화` : 스트림으로부터 데이터를 읽어서 객체를 만드는 것

### 7.1 ObjectInputStream, ObjectOutputStream
- `직렬화`에는 `ObjectOutputStream`을 사용, `역직렬화`에는 `ObjectInputStream` 사용
- InputStream/OutputStream을 상속받지만 기반스트림을 필요로하는 `보조스트림`이다.
- 객체를 생성할 때 입출력(직렬화/역직렬화)할 스트림을 지정해야한다.
```java
// 직렬화
// objectfile.ser파일로 통하는 파일 출력스트림 생성
FileOutputStream fos = new FileOutputStream("objectfile.ser");
// 파일 출력스트림의 직렬화 보조스트림생성
ObjectOutputStream out = new ObjectOutputStream(fos);
// UserInfo객체를 objectfile.ser파일에 저장
out.writeObject(new UserInfo());

// 역직렬화
// objectfile.ser파일로 통하는 파일 입력스트림 생성 (기반스트림)
FileInputStream fis = new FileInputStream("objectfile.ser");
// 파일 입력스트림의 역직렬화 보조스트림생성
ObjectInputStream in = new ObjectInputStream(fis);
// 저장된 데이터를 읽어서 저장
UserInfo info = (UserInfo)in.readObject();
```

- objectInputStream/objectOutputStream의 메서드   
: 직렬화/역직렬화를 `직접` 구현할 때 주로 사용

| ObjectInputStream | objectOutputStream |
|---|---|
| void defaultReadObject() | void defaultWriteObject() |
| int read() | void write(byte[] buf) |
| int read(byte[] buf, int off, int len) | void write(byte[] buf, int off, int len) |
| boolean readBoolean() | void write(int val) |
| byte readByte() | void writeBoolean(boolean val) |
| char readChar() | void writeByte(int val) |
| double readDouble() | void writeBytes(String str) |
| float readFloat() | void writeChar(int val) |
| int readInt() | void writeChars(String str) |
| long readLong() | void writeDouble(double val) |
| short readShort() | void writeFloat(float val) |
| Object readObject() | void writeInt(int val) |
| int readUnsignedByte() | void writeLong(long val) |
| int readUnsignedShort() | void writeObject(Object obj) |
| Object readUnshared() | void writeShort(int val) |
| String readUTF() | void writeUnshared(Object obj) |
|| void writeUTF(String str) |
> defalutReadObject() dafaultWriteObject()는 자동 직렬화를 수행

- `객체를 직렬화/역직렬화하는 작업`은 객체의 모든 인스턴스변수가 참조하고 있는 모든 객체에 대한 것이기 때문에 상당히 복잡하며 `시간이 오래 걸린다.` </br> 
: 직렬화 작업시간을 줄이고 싶으면 직렬화하고자 하는 객체의 클래스에 아래 두 개의 메서드 구현
```java
private void writeObject(ObjectOutputStream out) throws IOException {
    // write메서드를 사용해서 직렬화 수행
}

private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
    // read메서드를 사용해서 역직렬화 수행
}
```

### 7.2 직렬화가 가능한 클래스 만들기
- `Serializable`, `transient`
- 직렬화하고자하는 클래스가 `java.io.Serializable`인터페이스를 구현하도록하면 된다.
```java
// 인터페이스 정의
public interface Serializable { } // 아무런 내용도 없는 빈 인터페이스이지만 직렬화를 고려하여 작성한 클래스인지를 판단하는 기준이 된다.
```
- 조상이 Serializable을 구현하면 자손 클래스가 직렬화 할 수 있다.
```java
public class SuperUserInfo implements Serialozable {
    String name;
    String password;
}
public class UserInfo extends SuperUserInfo {
    int age;
}
// UserInfo 객체를 직렬화하면 조상 클래스의 name, password 인스턴스 변수도 직렬화 된다.
```

- Object클래스는 Serializable을 구현하고 있지 않기 때문에 Object객체는 직렬화할 수 없다.   
: `java.io.NotSerializableException` 발생
```java
// 직렬화 x
public class UserInfo implements Serializable {
    String name;
    String password;
    int age;

    Object obj = new Object();  // 예외발생
}

// 직렬화 가능
public class UserInfo implements Serializable {
    String name;
    String password;
    int age;

    Object obj = new String("abc"); // ok, 
    // 인스턴스변수 obj의 타입이 직렬화가 안되는 Object타입이긴 하지만 실제로 저장된 객체는 직렬화가 가능한 String인스턴스이기 때문에 직렬화 가능
}
```
> `주의` : 인스턴스 변수의 타입이 아닌 `실제로 연결된 객체의 종류`에 의해서 결정된다.

- `transient`제어자   
: 직렬화 대상에서 제외
```java
public class IserInfo implements Serializable {
    String name;
    transient String password;  // 직렬화 대상에서 제외
    int age;

    transient Object obj = new Object();    // 직렬화 대상에서 제외
}
```

### 7.3 직렬화가능한 클래스의 버전관리
- 직렬화된 객체를 역직렬화할 때는 직렬화 했을 때와 `같은 클래스`를 사용해야한다. (다를 경우 예외 발생)
- 객체가 직렬화될때 클래스에 정의된 멤버들의 정보를 이용해서 `serialVersionUID`라는 클래스의 버전을 `자동생성`해서 직렬화 내용에 포함된다.
- 단, `static변수`, `상수`, `transient`가 붙은 인스턴스변수는 직렬화에 영향을 주지않는다.
- 클래스의 버전이 바뀔 위험이 있는경우 클래스의 버전을 수동으로 관리해준다.
    + 클래스의 내용이 바뀌어도 클래스의 버전이 자동생성된 값으로 변경되지 않는다.
```java
class MyData implements java.io.Serializable {
    // 수동으로 관리
    static final long serialVersionUTD = 35187312039121231L;
    int value1;
}
```
- serialVersionUID 알아내는 방법
```java
> serialver 클래스이름  // serialVersionUID가 정의되어 있으면 정의된 값 출력, 정의되어있지 않으면 자동생성한 값 출력
```

# Reference
> 자바의 정석 - 남궁성
