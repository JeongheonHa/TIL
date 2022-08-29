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
| int read(byte[] b), int off, int len() | void write(byte[] b, int off, int len) |
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
| void srite(byte[] b, int off, int len) | 주어진 배열 b에 저장된 내용 중에서 off번째부터 len개 만큼만을 읽어서 출력소스에 쓴다. |
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

# Reference
> 자바의 정석 - 남궁성
