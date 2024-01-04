# LogExpress
## 목적
* 가볍고 단순한 Logger를 만들기 위한 프로젝트입니다.
* 비동기 로거입니다 - non-block 알고리즘을 사용합니다. Log4j2 와 logback 에서 지원하는 AsyncAppender 와 동일한 방식입니다. 원형 큐에 로그를 쌓았다가 백그라운드 스레드가 파일로 저장합니다.
* JAVA6 부터 호환됩니다. 

## 사용법
### gradle 의존성 추가
```groovy
dependencies {
    implementation 'io.github.clipsoft-rnd:LogExpressCore:0.10.6'
    // slf4j 1.7.36 기반 LogExpressSlf4j 0.10.6
    //implementation 'io.github.clipsoft-rnd:LogExpressSlf4j:0.10.6'
}
```
### 튜토리얼
* 다음 예제와 같이 사용할 수 있습니다. <br/>
  ```java
  import com.clipsoft.LogExpress.LogExpress;
  .
  .
  // default Logger 에 로그를 기록합니다. 
  LogExpress.trace("trace log");
  LogExpress.debug("debug log");
  LogExpress.info("info log");
  LogExpress.warn("debug log");
  LogExpress.error("error log");
  // 아래 코드와 위 코드와 동일하게 동작합니다.
  Logger baseLogger = LogExpress.newLogger();
  baseLogger.trace("trace log");
  baseLogger.debug("debug log");
  baseLogger.info("info log");
  baseLogger.warn("debug log");
  baseLogger.error("error log");
  // 임의의 marker 를 사용하여 로그를 기록할 수 있습니다.
  // 설정에 따라서 marker별로 다른 파일에 기록할 수 있습니다. 
  Logger markerLogger = LogExpress.newLogger("markerName");
  markerLogger.trace("trace log");
  markerLogger.debug("debug log");
  markerLogger.info("info log");
  markerLogger.warn("debug log");
  markerLogger.error("error log");			
  ```

* 로거 만들기
  ```java
	// 기본 로거
 	LogExpress.newLogger();
        // marker에 해당하는 로거 생성
        LogExpress.newLogger("marker");
        // Caller 클래스를 인자로 입력.
        // pattern 에 {caller} 를 사용할 경우 호출 클래스 이름을 출력합니다.
        LogExpress.newLogger(Caller.class);
        // Caller와 marker 함께사용
        LogExpress.newLogger(Caller.class, "marker");
  ```
<br/><br/>

### 설정 파일
* 설정 파일의 포맷은 ini을 따라가며 기본 파일명은 log-express.ini 입니다. 
* log-express.ini 파일은 리소스의 루트경로 혹은 프로젝트의 루트경로에 위치해야합니다. 
* 설정 파일의 경로를 변경하고 싶다면 jvm 옵션에 '-DLogExpress.configurationFile=파일경로' 를 추가하거나 프로세스 최초 시작지점에 아래와 같은 코드를 넣어주세요.
  ```java
    public static void main(String[] args) {
        System.setProperty("LogExpress.configurationFile", "파일경로/logexpress.ini");
        ... 
    }
    ```
* 아래는 설정 파일 예제와 각 옵션에 대한 자세한 설명입니다.

  ```ini
    #메인 설정
    [configuration]
    # LogExpress자체 문제를 확인할 수 있는 디버그 메시지를 출력하는 옵션입니다.
    # 기본값: false 
    debugMode.enable=false
    # debugMode.enable가 true일경우 LogExpress디버그 메시지를 파일로 기록합니다.
    # 로그 파일은 LogExpress의 jar라이브러리가 위치한 경로에 생성됩니다.
    # 기본값: false 
    debugMode.file=false
    
    # debugMode.enable가 true일경우 LogExpress디버그 메시지를 콘솔에 출력합니다.
    # 기본값: false 
    debugMode.console=false
    
    
    # 로그가 쌓이게될 원형 큐의 크기를 설정합니다.
    # 만약 큐가 가득 차게 된다면, 로그 기록 메서드를 호출하였을 때 큐에 비어있는 공간이 생길 때까지 대기합니다. 
    # 10보다 작은 값은 설정할 수 없습니다.
    # 기본값: 128000
    queueSize=128000
    
    # non-blocking알고리즘(CAS)을 사용하는 큐에 로그를 쌓습니다.
    # false를 사용하면 뮤텍스 락을거는 큐를 사용합니다. CAS알고리즘이 잘 동작하지 않는 환경에서 false값을 사용할 수 있습니다. 
    # 기본값: true
    nonBlockingQueue=true
    
    
    # 메인 스레드가 종료되었다면, 로그 큐의 내용이 모두 비워지고 나서 로거 스레드도 종료합니다.
    # 일반적인 JVM 환경에서만 동작합니다. (메인 스레드의 id 가 1일 경우) 
    # 다중 스레드를 이용하는 서버 운영 환경에서는 사용을 권장하지 않습니다.
    # 기본값: false 
    autoShutdown=false
    
    
    
    

    # true로 설정하면 main 스레드가 종료될때 로거 스레드도 함께 종료합니다.
    # 기본값: false
    daemonThread=false
    
    # 기본 marker 를 설정합니다. marker 는 tag의 개념과 같습니다.
    # defaultLogger() 메서드를 호출하였을 때 해당 marker 를 갖고 있는 Logger 객체를 가져옵니다.
    # 만약 지정하지 않는다면 첫 번째 writer 옵션의 이름이 기본 marker 가 됩니다.
    # 기본값: 비어있음
    defaultMarker=api
    
    # 비어있는 로그 큐를 검사하는 간격을 밀리세컨드 단위로 설정합니다. 
    # 만약 큐가 비어있는 경우 이 옵션 값 만큼 대기(wait)를 하게 됩니다.
    # 0이하의 값을 설정하면 비어있는 로그 큐를 검사하지 않습니다.
    # 기본값: 3000
    workerInterval=3000
    
    # 항상 파일 존재 여부를 확인하고 없으면 다시 생성합니다. 
    # 기본값: false 
    fileExistCheck=true
    
    
    # writer 는 logback 의 appender와 유사합니다.  
    # writer 옵션. Logger의 객체는 해당 marker의 Writer를 사용합니다. 만약 지정되지 않은 marker를 사용할 경우 기본 Writer를 사용하게됩니다.
    # writer 의 섹션 이름은 항상 'writer/' 로 시작해야 합니다. 그 뒤에 오는 것은 marker 입니다. 이 예제에서 marker는 'api'입니다. 
    [writer/api]
    # 같은 Writer 옵션을 공유하는  marker 를 추가할 수 있습니다. 쉼표로 구분됩니다.
    markers=catalina,bootstrap
    
    # level 을 설정합니다.
    # trace, debug, info, warn, error, off 순으로 레벨이 낮아집니다. off 레벨에서는 어떤 로그도 출력하지 않습니다.
    # 기본값: info
    level=info
    
    # 버퍼 크기를 byte단위로 설정합니다. 
    bufferSize=1024
    
    # 파일의 최대 크기를 Mib 단위로 설정합니다.
    # file옵션에 {number} 가 있어야 작동합니다. 
    # 0이하로 설정하거나 file옵션에 {number} 가 없으면 무제한 크기를 갖습니다.
    # 기본값: 512mib
    maxSize=512
    
    # 최대 기록 날짜를 설정합니다.
    # 설정한 날짜가 지난 로그 파일은 자동으로 삭제됩니다. 
    # file옵션에 {date::} 가 있어야 작동합니다. date format 에 따라서 오동작할 수 있으니 잘 설정하세요. 
    # 0보다 작은 값으로 설정시 기간 제한이 없습니다.
    # 기본값: 60일
    maxHistory=60
    
    # 파일 경로 및 패턴을 설정합니다.
    #  - {marker}: marker 이름
    #  - {hostname}: 호스트이름
    #  - {pid}:  프로세스 id. 사용을 권장하지 않음. 
    #  - {date::(date format)}: 로그 기록 날짜를 설정.
    #  - {number}: maxSize 옵션에 따라서 파일 크기가 최대치에 도달하면, 새로운 번호의 파일로 생성됩니다.
    file={marker}.{hostname}.{date:yyyy-MM-dd}.{number}.txt
    
    # 로그 라인 패턴을 설정합니. 
    #  - {time::(date format)}: 로그 발생 시간
    #  - {level}: 로그 레벨
    #  - {hostname}: 호스트 이름
    #  - {pid}: 프로세스 id
    #  - {thread}: 스레드 이름
    #  - {tid}: 스레드 id
    #  - {marker}: marker 이름
    #  - {message}: 로그 메시지
    # !주의! 아래의 옵션을 추가시에 로그 속도가 약 0.3배로 떨어지며 더 많은 메모리를 사용합니다. 
    #  - {file}: 로그가 발생된 파일명
    #  - {class}: 로그가 발생된 클래스의 full package 경로
    #  - {class-name}: 로그가 발생된 클래스명 (패키지 경로 제외)
    #  - {method}: 로그가 발생된 메서드명
    #  - {line}: 로그가 발생된 라인번호
    #  - {caller}: 로그를 호출한 클래스. LogExpress.getLogger 메서드의 인자를 통하여 정의됨.
    #  - {caller-simple}: 로그를 호출한 클래스의 패키지 경로를 제외한 이름
    pattern={time::HH:mm:ss.SSS} [{level}] <{hostname}/PID:{pid}/{thread}:{tid}>  {marker}  | {caller} {caller-simple}  | ({file}) {class-name}.{method}():{line} | {message};
    
    # 로그 기록 타입을 설정합니다.
    # file, console 타입을 지정할 수 있습니다.
    # 기본값: console 
    types=file, console
    
    
    # encoding 을 설정합니다.
    # 기본값: 비어있음. 시스템 기본 encoding 사용. 
    encoding=
    
    
    # 만약 Looger 를 감싸는 facade 나 wrapper 클래스를 만들 경우, 로그 호출위치를 더 정확하게 명시할 수 있습니다.
    # 로그에 출력되는 클래스 및 메서드, 라인번호가 실제 호출한 위치와 다르다면 이 값을 조정하세요.
    # 기본값: 1
    stackTraceDepth=1
    ```

### 코드상에서 동적으로 설정 변경

* 동적으로 설정 변경이 가능합니다. 로그 큐와 Worker 를 모두 종료시키고 새로 생성된 객체로 안전하게 교체(exchange)합니다.
  ```java
    // 기존 설정을 복제합니다.
    Configuration config = LogExpress.cloneConfiguration();
    // 빈 설정을 생성합니다.
    // Configuration config  = Configuration.newConfiguration();
    // 기본 위치의 설정 파일을 읽어옵니다.
    //Configuration config  = Configuration.fromDefaultConfigurationFile();
    // 경로를 지정하여 설정 파일을 읽어옵니다.
    //Configuration config  = Configuration.newConfiguration("../conf/log.ini");
    // 경로를 지정하여 설정 파일을 읽어옵니다.						
    config.clearWriters();
    WriterOption option = config.newWriterOption("api");
    option.setFile("./log.{hostname}.{marker}.{date:yyyy-MM-dd}.{number}.txt");
    config.setDefaultMarker("api");
    config.setQueueSize(100000000);
    // 변경된 설정을 업데이트합니다.
    LogExpress.updateConfig(config);
  ```
### 환경변수 및 프로퍼티 적용

* 문자열로 입력 가능한 모든 설정 값에 환경변수 혹은 프로퍼티 값을 넣을 수 있습니다.
* 환경 변수는 '%환경 변수 키%' 로 정의되며 프로퍼티는 '${프로퍼티 키}' 와 같은 형식으로 설정됩니다. 만약 해당 키에 해당하는 값이 없다면 설정 문자열이 그대로 전체 값에 들어갑니다.
  ```ini
  # 사용 예 - System.properties 에 임의로 정의된 Log.dir 값과 시스템 환경변수 USER 에 해당하는 값
  file=${Log.dir}/%USER%/%log.out
  ```
* 기본 프로퍼티 값
  * ${LogExpress.path}    : LogExpress 라이브러리가 위치한 경로. 즉, root 경로.
  * ${LogExpress.hostname} : 호스트네임
  * ${LogExpress.pid} : 프로세스 아이디

### 메시지 패턴 요소별 길이 제한
* 메시지 패턴의 각 요소에 길이 제한을 걸 수 있습니다.
* 대괄호 안에 [최소길이:최대길이] 를 입력하여 제한합니다.
* 텍스트의 길이가 최소길이보다 작을 경우 정렬되는 위치를 정할 수 있습니다. 대괄호 안쪽의 공백을 이용합니다. 여는 대괄호 바로 뒤에 공백이 있다면 우측정렬([공백), 양쪽다 공백이 있다면 중앙정렬. 기본은 좌측정렬입니다.
* 최소길이 혹은 최대길이만 정의할 수 있습니다.
* 최대길이에서 마이너스(-) 값을 사용하면 텍스트를 뒤에서부터 설정된 길이만큼 출력합니다.
  ```ini
   # 메시지 길이를 최소 5자, 최대 5자로 동일하게 출력.
   pattern={time::HH:mm:ss.SSS} {message[5:5]}
   # 메시지 길이를 최소 10자로 설정. 만약 메시지가 10자 미만이면 좌우 공백을 줘서 중앙정렬
   pattern={time::HH:mm:ss.SSS} {message[ 10: ]}
   # 메시지를 뒤에서부터 5자만 출력
   pattern={time::HH:mm:ss.SSS} {message[:-5]} 
  ```

### 로거와 프로세스 종료
* 비동기 로거인 LogExpress는 메인 스레드가 종료되어도 로거 스레드는 종료되지 않습니다. 로거 스레드가 존재하는한 프로세스도 종료되지 않을 것입니다. 만약 로거 스레드와 프로세스를 우아하게 종료하고싶다면 LogExpress.shutdown() 을 사용하세요.
  ```java
  ShutdownFuture future = LogExpress.shutdown();
  // 여러개의 이벤트 등록가능
  future.setOnEndCallback(new Runnable() {
	@Override
	public void run() {
		System.out.println("끝1");
	}
   });

  future.setOnEndCallback(new Runnable() {
	@Override
	public void run() {
		System.out.println("끝2");
	}
  });
  // 로거 스레드가 끝날때까지 대기
  furture.await();
  ```
  * 설정파일 log-express.ini 에서 autoShutdown 의 값을 true로 변경하여 로거를 자동으로 종료되도록 할 수 있습니다. 하지만 다중 스레드를 이용하는 서버 운영 환경에서는 사용을 권장하지 않습니다.
   

### 메모리를 절약하고 빠르게 사용하기 위한 팁
* debug 로그 레벨 이상은 isDebug() 를 사용하여 로그 레벨을 검사하세요.
  ```java
  if(LogExpress.isDebugEnabled()) {
     LogExpress.debug("소켓 문제가 발생하였습니다.", exception);
  }
  ```
   
