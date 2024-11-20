# 이메일 발송 기능 비동기 처리 적용기

크루루 서비스의 이메일 발송 기능에 비동기 처리를 적용하며 마주한 문제와 그 해결 과정을 소개합니다.  
이 글은 이메일 발송 기능 구현 자체보다는 비동기 처리의 필요성과 이를 적용하는 과정에서 발생한 문제와 해결 방안을 다룹니다.

아래의 선행 지식이 뒷받침된다면 내용을 이해하기 더욱 쉽습니다.
- Spring에서의 @Transactional 어노테이션과 트랜잭션 관리
- 자바에서의 비동기 프로그래밍(@Async 및 CompletableFuture)
- MultipartFile를 활용한 파일 처리
- Mockito를 이용한 테스트 코드 작성

## 초기 코드의 문제점

우리가 원하는 기능은 이메일 발송 성공 시에만 발송 내역을 저장하는 것이었습니다.

아래와 같이 EmailFacade에 두 로직을 한 메서드로 묶고 @Transactional을 적용하여, 이메일 발송 실패 시 이메일 발송 내역 저장 로직이 롤백되도록 구현하였습니다.
```java
// EmailFacade.sendAndSave()
@Transactional
public void sendAndSave(Club from, Applicant to, String subject, String content, List<MultipartFile> files) {
    emailService.send(from, to, subject, content, files);
    emailService.save(from, to, subject, content);
}
```
그러나 이 코드는 다음과 같은 문제를 야기했습니다.

**첫 번째, 외부 API 통신을 DB 트랜잭션 범위 안에서 수행한다.**

일반적으로 이메일 전송과 같이 우리가 제어할 수 없는 외부 API 통신은 DB 트랜잭션 내에서 제외하는 것이 좋습니다.
트랜잭션을 처리하기 위해서는 DB 커넥션이 필요한데, 외부 API의 응답을 기다리는 동안 해당 커넥션이 점유되면 병목 현상이 발생할 위험이 커지기 때문입니다.

보통 이에 대한 개선책으로 [**퍼사드 패턴**](https://ko.wikipedia.org/wiki/%ED%8D%BC%EC%82%AC%EB%93%9C_%ED%8C%A8%ED%84%B4)이 제시됩니다.
크루루 서비스는 이미 모든 도메인에 이 패턴을 전역적으로 적용하여 코드의 복잡도를 줄이고 유지보수를 용이하게 하고자 했습니다.
또한, 외부 API를 활용한 기능을 구현할 때 앞서 말한 문제점을 예방하는 목적도 있었습니다.  
그러나 위 코드와 같이 외부 API 호출이 트랜잭션 범위에 포함되면서, 결국 퍼사드 패턴의 이점을 제대로 활용하지 못했습니다.

**두 번째, 클라이언트의 응답 대기 시간이 길어져 사용자 경험의 저하가 우려된다.**

이메일 발송이 완료될 때까지 기다려야 하므로 클라이언트의 응답 대기 시간이 길어지는 문제가 발생했습니다.
고작 한 명에게 메일을 발송하는 데도 꽤 긴 시간이 걸리는데, 다수의 사용자에게 메일을 발송한다면 클라이언트의 대기 시간이 굉장히 길어질 것으로 예상하였습니다.
이로 인해 사용자 경험의 저하로 이어질 위험이 있었습니다.

---

## 해결 방안 1. EventListener와 @Async 사용하기

첫 번째로 고안해 낸 방안은 다음과 같습니다.

- 첫 번째 문제: 트랜잭션 완료 후 이메일 발송을 별도의 EventListener에서 처리
- 두 번째 문제: 이메일 발송 작업에 비동기 처리 적용

아래는 해당 방안을 구현한 코드의 일부이며, 아직 비동기 처리는 적용되지 않은 상태입니다.

- SendEmailEvent

    ```java
    @Getter
    public class SendEmailEvent extends ApplicationEvent {
    
        private Club from;
        private Applicant to;
        private String subject;
        private String content;
        private List<MultipartFile> files;
    
        public SendEmailEvent(
                Object source,
                Club from,
                Applicant to,
                String subject,
                String content,
                List<MultipartFile> files
        ) {
            super(source);
            this.from = from;
            this.to = to;
            this.subject = subject;
            this.content = content;
            this.files = files;
        }
    }
    ```

- EmailEventListener

    ```java
    @Component
    @RequiredArgsConstructor
    public class EmailEventListener {
    
        private final EmailService emailService;
    
        @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT, classes = SendEmailEvent.class)
        public void handleSendEmailEvent(SendEmailEvent event) {
            emailService.send(event.getFrom(), event.getTo(), event.getSubject(), event.getContent(), event.getFiles());
        }
    }
    ```

- EmailFacade

    ```java
    @Service
    @Transactional(readOnly = true)
    @RequiredArgsConstructor
    public class EmailFacade {
    
        private final EmailService emailService;
        private final ClubService clubService;
        private final ApplicantService applicantService;
        private final ApplicationEventPublisher eventPublisher;
    
        @Transactional
        public void send(EmailRequest request) {
            Club from = clubService.findById(request.clubId());
            request.applicantIds()
                    .stream()
                    .map(applicantService::findById)
                    .forEach(to -> sendAndSave(from, to, request.subject(), request.content(), request.files()));
        }
    
        @Transactional
        public void sendAndSave(Club from, Applicant to, String subject, String content, List<MultipartFile> files) {
            emailService.save(from, to, subject, content);
            eventPublisher.publishEvent(new SendEmailEvent(this, from, to, subject, content, files));
        }
    }
    
    ```


@TransactionalEventListener를 사용하여 트랜잭션이 성공한 경우에만(= 발송 내역이 성공적으로 저장된 경우에만) 메일을 발송할 수 있게 하였고,
트랜잭션이 실패해도 메일이 발송되어 버리는 문제를 방지하였습니다.

그러나 결국 이 방안을 도입하지 않았습니다. 같은 팀원인 명오의 아주 예리한 지적 때문이었습니다.

> 이렇게 구현해도 EventListener너를 적용한 코드와 결과가 같지 않나?
>

```java
public void sendAndSave(Club from, Applicant to, String subject, String content, List<MultipartFile> files) {
    emailService.save(from, to, subject, content);
    emailService.send(from, to, subject, content, files);
}
```

맞습니다. 이 코드의 경우 save()에서 예외가 발생하면 이후 코드는 실행되지 않습니다. 즉, 이메일이 발송되지 않는다는 것입니다.
결과가 동일하다면 더 간단한 방법을 선택하는 것이 더 합리적이기에, 코드의 복잡도를 올리는 EventListener를 걷어냈습니다.

---

## 해결 방안 2. @Async와 CompletableFuture 사용하기

### @Async의 한계

비동기 처리를 적용하기 위해 처음에 생각해 낸 방법은 @Async 어노테이션을 사용하는 것이었습니다.

아래와 같이 단순하게 이메일을 발송하는 로직에만 @Async 어노테이션을 붙여주었습니다.

- EmailService

    ```java
    
        @Async
        public void send(Club from, Applicant to, String subject, String content, List<MultipartFile> files) {
            try {
                System.out.println("비동기 메서드 실행 시작...");
                MimeMessage message = mailSender.createMimeMessage();
                MimeMessageHelper helper = new MimeMessageHelper(message, true);
                helper.setTo(to.getEmail());
                helper.setSubject(subject);
                helper.setText(content);
                if (hasFile(files)) {
                    addAttachments(helper, files);
                }
                mailSender.send(message);
                System.out.println("비동기 메서드 실행 완료...");
            } catch (Exception e) {
                // ...
            }
        }
        
        ...
        
        @Transactional
        public void save(Club from, Applicant to, String subject, String content, List<MultipartFile> files) {
            Email email = new Email(from, to, subject, content, true);
            emailRepository.save(email);
        }
    ```

- EmailFacade

    ```java
        public void send(EmailRequest request) {
            Club from = clubService.findById(request.clubId());
            request.applicantIds()
                    .stream()
                    .map(applicantService::findById)
                    .forEach(to -> sendAndSave(from, to, request.subject(), request.content(), request.files()));
        }
    
        public void sendAndSave(Club from, Applicant to, String subject, String content, List<MultipartFile> files) {
            System.out.println("이메일 발송 시작...");
            emailService.send(from, to, subject, content, files);
            System.out.println("이메일 발송 완료...");
            emailService.save(from, to, subject, content, files);
            System.out.println("이메일 발송 내역 저장 완료");
        }
    ```

- 출력 결과

    ```
    Hibernate: select c1_0.club_id,c1_0.member_id,c1_0.name from club c1_0 where c1_0.club_id=?
    Hibernate: select a1_0.applicant_id,a1_0.created_date,a1_0.email,a1_0.is_rejected,a1_0.name,a1_0.phone,a1_0.process_id,a1_0.updated_date from applicant a1_0 where a1_0.applicant_id=?
    이메일 발송 시작...
    이메일 발송 완료...
    비동기 메서드 실행 시작...
    Hibernate: insert into email (content,created_date,club_id,is_succeed,subject,applicant_id,updated_date) values (?,?,?,?,?,?,?)
    이메일 발송 내역 저장 완료
    Hibernate: select a1_0.applicant_id,a1_0.created_date,a1_0.email,a1_0.is_rejected,a1_0.name,a1_0.phone,a1_0.process_id,a1_0.updated_date from applicant a1_0 where a1_0.applicant_id=?
    이메일 발송 시작...
    이메일 발송 완료...
    비동기 메서드 실행 시작...
    Hibernate: insert into email (content,created_date,club_id,is_succeed,subject,applicant_id,updated_date) values (?,?,?,?,?,?,?)
    이메일 발송 내역 저장 완료
    비동기 메서드 실행 완료...
    비동기 메서드 실행 완료...
    ```


출력되는 로그 순서를 봤을 때 비동기 메서드의 완료 여부와 상관없이 다음 작업을 수행하는 것을 보아, 이메일 발송 로직에 비동기가 잘 적용되었음을 확인할 수 있었습니다.

하지만 이 방식에도 문제점이 있었습니다. 이메일 발송 성공 여부와 상관없이 발송 내역이 저장된다는 점입니다.

'이메일 발송 작업', 즉 '비동기 작업'의 성공 여부를 확인할 수 있는 장치가 필요했습니다.
이를 위해 EmailService.send() 메서드가 발송 결과를 포함한 Email 객체를 반환하고,
그 객체를 발송 내역 테이블에 저장해 주는 방법을 떠올렸습니다.

문제는 @Async만으로는 이 방법을 적용할 수 없었습니다. @Async가 적용된 메서드는 void를 반환해야 한다는 제약때문이었습니다.
즉, 이메일 발송 작업의 결과를 직접 반환할 수 없었습니다.

### CompletableFuture 이용

CompletableFuture는 자바에서 제공하는 비동기 프로그래밍 도구로, 비동기 작업의 결과를 유연하게 처리할 수 있는 특징이 있습니다.

이를 통해 앞서 말한 `EmailService.send()가 발송 결과로 CompletableFuture<Email> 객체를 반환하고,
해당 객체를 발송 내역 테이블에 저장하는 로직`을 구현하였습니다. 

- EmailService

    ```java
        @Async
        // 이메일 발송 결과로 CompletableFuture<Email> 객체를 반환합니다.
        public CompletableFuture<Email> send(
                Club from, Applicant to, String subject, String content, List<MultipartFile> files) {
            try {
                System.out.println("비동기 메서드 실행 시작...");
                MimeMessage message = mailSender.createMimeMessage();
                
                ...
  
                mailSender.send(message);
                System.out.println("비동기 메서드 실행 완료...");
                // Email(from, to, subject, content, isSucceed)
                // 이메일 발송에 성공한 경우 Email 엔티티의 isSucceed 필드를 true로 설정합니다.
                return CompletableFuture.completedFuture(new Email(from, to, subject, content, true));
            } catch (MessagingException | MailException e) {
                // 이메일 발송에 실패한 경우 Email 엔티티의 isSucceed 필드를 false로 설정합니다.
                return CompletableFuture.completedFuture(new Email(from, to, subject, content, false));
            }
        }
    ```

    - 이메일 발송 결과로 CompletableFuture<Email> 객체를 반환합니다.
    - 이메일 발송에 성공한 경우 Email 엔티티의 isSucceed 필드를 true로, 실패한 경우에는 false로 설정합니다.


- EmailFacade

    ```java
        public void send(EmailRequest request) {
            Club from = clubService.findById(request.clubId());
            List<Applicant> applicants = request.applicantIds()
                    .stream()
                    .map(applicantService::findById)
                    .toList();
            sendAndSave(from, applicants, request.subject(), request.content(), request.files());
        }
    
        private void sendAndSave(Club from, List<Applicant> tos, String subject, String text, List<MultipartFile> files) {
            System.out.println("sendAndSave 시작...");
            tos.stream()
                    .map(to -> emailService.send(from, to, subject, text, files))
                    .map(future -> future.thenAccept(emailService::save))
                    .toList();
            System.out.println("sendAndSave 완료...");
        }
    ```

    - CompletableFuture가 완료될 때마다 emailService.save() 메서드를 호출하여 발송 내역을 저장합니다. thenAccept()는 비동기 작업이 성공적으로 완료되었을 때 후속 작업을 정의합니다.


- 출력 결과
    ```
    sendAndSave 시작...
    비동기 메서드 실행 시작...
    sendAndSave 완료...
    비동기 메서드 실행 시작...
    비동기 메서드 실행 완료...
    이메일 발송 내역 저장 시작...
    Hibernate: insert into email (content,created_date,club_id,is_succeed,subject,applicant_id,updated_date) values (?,?,?,?,?,?,?)
    이메일 발송 내역 저장 완료...
    비동기 메서드 실행 완료...
    이메일 발송 내역 저장 시작...
    Hibernate: insert into email (content,created_date,club_id,is_succeed,subject,applicant_id,updated_date) values (?,?,?,?,?,?,?)
    이메일 발송 내역 저장 완료...
    ```

  비동기 메서드 작업이 완료된 후, 즉 이메일 발송 작업이 완료된 후에야 발송 내역을 저장하는 것을 확인할 수 있습니다.


---

## 또 다른 난관에 봉착 
### 난관 1. NoSuchFileException

이메일 발송 시 파일 첨부 기능을 위해 MultipartFile을 사용하였습니다.

그러나 비동기 처리를 적용하고 나니 첨부 파일이 있는 이메일을 발송할 때마다 `java.nio.file.NoSuchFileException`이 발생했습니다.

```java
Failed messages: jakarta.mail.MessagingException: IOException while sending message;
  nested exception is:
		java.nio.file.NoSuchFileException: /private/var/folders/q9/256wydx95tbgjsx737_3zysc0000gp/T/tomcat.8080.*/upload_*.tmp
```

이는 MultipartFile이 임시 저장소에 파일을 저장하는 특징 때문이었습니다.

크루루 서비스에서는 application.yml 파일에 아래와 같이 설정을 해주었습니다.

```yaml
servlet:
    multipart:
      enabled: true
      file-size-threshold: 2KB
      max-file-size: 25MB
      max-request-size: 50MB
```

file-size-threshold를 2KB로 설정했기 때문에, 첨부 파일이 2KB 초과하면 임시 저장소에 저장됩니다.
사실상 대부분의 파일이 메모리가 아닌 임시 저장소에 저장되는 구조입니다.

문제는 임시 저장소에 저장된 파일이 HTTP 요청이 종료되거나 처리 메서드가 종료되면 자동으로 삭제된다는 점입니다.
따라서 비동기적으로 이메일을 발송할 때 이미 파일이 삭제되어 더 이상 접근할 수 없게 되고 NoSuchFileException이 발생합니다.

해결 방법은 간단합니다. 비동기 작업 시작 전에 임시 파일을 영구 저장하고, 작업 완료 후 명시적으로 삭제하면 됩니다.

크루루 서비스는 아래와 같이 EmailFacade와 FileUtil에 임시 파일을 저장하는 메서드를 만들어서 이 문제를 해결하였습니다. 
비동기 작업 전에 파일을 저장하고, 완료 후 파일을 명시적으로 삭제했습니다.

```java
// EmailFacade.saveTempFiles()
private List<File> saveTempFiles(Club from, String subject, List<MultipartFile> files) {
    try {
        return FileUtil.saveTempFiles(files);
    } catch (IOException e) {
        throw new EmailAttachmentsException(from.getId(), subject);
    }
}

// FileUtil.saveTempFiles()
public static List<File> saveTempFiles(List<MultipartFile> files) throws IOException {
    if (files == null) {
        return new ArrayList<>();
    }
    List<File> tempFiles = new ArrayList<>();
    for (MultipartFile file : files) {
        File tempFile = File.createTempFile(FILE_PREFIX, FILE_SUFFIX + file.getOriginalFilename());
        file.transferTo(tempFile);
        tempFiles.add(tempFile);
    }
    return tempFiles;
}

// FileUtil.deleteFiles()
public static void deleteFiles(List<File> files) {
    if (files != null) {
        files.forEach(FileUtil::deleteFile);
    }
}

// FileUtil.deleteFile()
private static void deleteFile(File file) {
    if (file.exists()) {
        file.delete();
        return;
    }
    log.info("삭제할 파일이 존재하지 않습니다: {}", file.getAbsolutePath());
}
```

```java
// EmailFacade.sendAndSave()
private void sendAndSave(Club from, List<Applicant> tos, String subject, String text, List<MultipartFile> files) {
    List<File> tempFiles = saveTempFiles(from, subject, files);
    
    List<CompletableFuture<Void>> futures = tos.stream()
        .map(to -> emailService.send(from, to, subject, text, files))
        .map(future -> future.thenAccept(emailService::save))
        .toList();
    
    CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
        .thenRun(() -> FileUtil.deleteFiles(tempFiles));
}
```

---

### 난관 2. 비동기 테스트 작성

앞에서 했던 것처럼 출력문을 일일이 넣어 메서드들의 실행 순서를 확인하는 방식은 매우 비효율적입니다.

이메일 발송 작업이 비동기적으로 올바르게 동작하는지 효율적으로 검증하기 위해 테스트 코드를 작성하였습니다.
이를 위해 시도한 방식들을 차례로 소개합니다.
<br><br>


#### 시도 1: 메서드 호출 전후 시각의 차이로 비동기 판단

```java
    @DisplayName("이메일을 비동기로 발송한다.")
    @Test
    void sendEmailMany() throws IOException {
        // given
        // javaMailSender.send()가 호출될 때마다 1초의 지연을 주어
        // 실제 이메일이 비동기로 발송되는 것처럼 보이도록 합니다. 
        Mockito.doAnswer(invocation -> waitSeconds(1))
                .when(javaMailSender).send(any(MimeMessage.class));
        
        ...테스트 데이터 생성...
        
        EmailRequest emailRequest = new EmailRequest(...);

        // when
        long before = System.currentTimeMillis();
        emailFacade.send(emailRequest);
        long after = System.currentTimeMillis();

        // then
        // 전송 완료까지 걸린 시간이 1초보다 짧은지 검증합니다.
        // 만약 비동기로 처리되었다면, 메서드는 즉시 반환되어야 하므로 테스트는 통과합니다.
        assertThat(after - before).isLessThan(1000);
    }
    
    private Object waitSeconds(long timeout) {
        try {
            TimeUnit.SECONDS.sleep(timeout);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return null;
    }
```

emailFacade.send() 메서드의 호출 전후의 시각을 기록하여 두 시각의 차이에 의존하여 비동기 처리가 제대로 이뤄졌는지 판단하는 방식입니다.

그러나 이 방식으로는 이메일 발송과 저장이 실제로 수행되었는지를 확인할 수 없습니다.
<br><br>

#### 시도 2: Mockito.verify()를 활용하여 이메일 발송 및 발송 내역 저장 검증

```java
    @DisplayName("이메일을 비동기로 발송하고, 발송 내역을 저장한다.")
    @Test
    void sendAndSave() {
        // given
        // javaMailSender.send()가 호출될 때마다 1초의 지연을 주어
        // 실제 이메일이 비동기로 발송되는 것처럼 보이도록 합니다.
        Mockito.doAnswer(invocation -> waitSeconds(1))
                .when(javaMailSender).send(any(MimeMessage.class));
                
        ...테스트 데이터 생성...
        
        EmailRequest request = new EmailRequest(...);

        // when
        emailFacade.send(request);

        // then
        verify(javaMailSender, times(0)).send(any(MimeMessage.class));
        waitSeconds(2);
        verify(javaMailSender, times(1)).send(any(MimeMessage.class));
        verify(emailService, times(1)).save(any(Email.class));
    }
    
    private Object waitSeconds(long timeout) {
        try {
            TimeUnit.SECONDS.sleep(timeout);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return null;
    }
```

Mockito.verify() 메서드를 사용하여 이메일 발송과 발송 내역 저장 작업을 각각 검증하는 방식입니다. 이로써 `시도 1`에서 언급한 문제는 해결하였습니다.

그러나 이 방식은 테스트가 오래 걸린다는 단점이 남아있습니다. TimeUnit.SECONDS.sleep(2)를 사용하면 최소 2초 동안은 반드시 대기해야 합니다.
<br><br>

#### 시도 3: Awaitility 이용

```java
    @DisplayName("이메일을 비동기로 발송하고, 발송 내역을 저장한다.")
    @Test
    void sendAndSave() {
        ...

        // when
        emailFacade.send(request);

        // then
        verify(javaMailSender, times(0)).send(any(MimeMessage.class));
        await().atMost(2, TimeUnit.SECONDS).untilAsserted(() -> {
            verify(javaMailSender, times(1)).send(any(MimeMessage.class));
            verify(emailService, times(1)).save(any(Email.class));
        });
    }
```

Awaitility를 사용하면 설정한 시간 내에 특정 조건을 만족할 때까지만 대기하도록 구현할 수 있습니다.
위 코드에서 `await().atMost(2, TimeUnit.SECONDS).untilAsserted(() -> { … });` 처럼 작성하면,
2초 이내에 javaMailSender.send()와 emailService.save() 메서드가 각각 한 번씩 호출되면 검증이 바로 완료됩니다.
이메일이 발송되고 발송 내역이 저장되는 작업이 1초 만에 끝나면, 남은 시간 동안 기다리지 않고 즉시 검증을 종료합니다.

결과적으로 이메일 발송 및 발송 내역 저장 작업의 완료 여부를 효율적으로 확인하면서 불필요한 대기를 줄일 수 있게 되었습니다.

---

## 마무리
지금까지 이메일 발송 기능에 비동기 처리를 적용하며 겪은 문제들과 해결 방안을 소개해 드렸습니다. 
트랜잭션 내에서 외부 API를 분리하여 성능 저하와 병목 현상을 방지했고, 
비동기 처리를 적용하여 성능 최적화와 사용자 경험 개선이라는 두 가지 목표를 달성할 수 있었습니다.


또한, 이메일 발송 성공 여부를 저장해야 하는 요구사항을 만족하기 위해 CompletableFuture와 @Async를 활용한 비동기 처리를 구현했으며,
이 과정에서 발생한 임시 파일 처리 문제까지 해결하였습니다. 

향후 애플리케이션의 더 나은 성능, 그리고 더욱 확장성있는 아키텍처를 위해 메시지 큐 도입을 검토해 볼 예정입니다.
이를 통해 더 복잡한 비동기 작업도 효과적으로 관리하여, 안정성과 확장성을 갖춘 서비스를 구축할 수 있을 것으로 기대합니다.

---

## 참고 자료

- [트랜잭션 내에 외부 리소스 요청이 담기게 되면 어떤 문제가 발생할까?](https://tecoble.techcourse.co.kr/post/2022-09-20-external-in-transaction/)
- [퍼사드 패턴](https://ko.wikipedia.org/wiki/%ED%8D%BC%EC%82%AC%EB%93%9C_%ED%8C%A8%ED%84%B4)
- [Java Docs - CompletableFuture](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/CompletableFuture.html)
- [모던 자바 인 액션 - CHAPTER 16 CompletableFuture : 안정적 비동기 프로그래밍](https://www.hanbit.co.kr/store/books/look.php?p_code=B4926602499)
- [MultipartFile](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/multipart/MultipartFile.html)
- [Why do I get NoSuchFileException when using multipartFile @Async in java spring boot?](https://stackoverflow.com/questions/77620046/why-do-i-get-nosuchfileexception-when-using-multipartfile-async-in-java-spring)
- [Awaitility](http://www.awaitility.org/)
