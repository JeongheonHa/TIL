# Commit Message의 7가지 규칙
> 모두가 알아볼 수 있는 좋은 커밋메서지는 어떻게 작성해야하는지 알아보자
 1. 제목과 본문을 빈 행으로 구분한다   
 2. 제목을 50글자 내로 제한   
 3. 제목 첫 글자는 대문자로 작성   
 4. 제목 끝에 마침표 넣지 않기   
 5. 제목은 명령문으로 사용하며 과거형을 사용하지 않는다   
 6. 본문의 각 행은 72글자 내로 제한   
 7. 어떻게 보다는 무엇과 왜를 설명한다   

# Commit Message의 구조
```html
<type>(<scope>): <subject>          -- 헤더 (필수)
<BLANK LINE>
<body>                              -- 본문 (선택)
<BLANK LINE>
<footer>                            -- 바닥글 (선택)
```

```
# <type>의 성격 (아래 중 하나여야한다)

feat     : 새로운 기능에 대한 커밋
fix      : 버그 수정에 대한 커밋
build    : 빌드 관련 파일 수정에 대한 커밋
chore    : 그 외 자잘한 수정에 대한 커밋
ci       : CI관련 설정 수정에 대한 커밋
docs     : 문서 수정에 대한 커밋
style    : 코드 스타일 혹은 포맷 등에 관한 커밋
refactor : 코드 리팩토링에 대한 커밋
test     : 테스트 코드 수정에 대한 커밋
```

```
# <body>
header만으로는 표현할 수 없는 세부적인 내용으 적는다.
```

```
# <footer>
참조 정보를 추가하는 용도로 사용한다.
```

# 커밋 메세지로 Github issue 자동 종료시키는 방법

## 1. Supported keywords
- close
- closes
- closed
- fix
- fixes
- fixed
- resolve
- resolves
- resolved
```
keyword # issue number  // 커밋 메세지의 어느 위치에서든 사용 가능
```
## 2. 각각의 키워드 마다 기능의 차이는 없지만 문법과 문맥에 맞춰 사용하자   
- close 계열 - 일반 개발 이슈   
- fix 계열 - 버그 픽스, 핫 픽스 이슈   
- resolve 계열 - 문의, 요청 사항에 대응한 이슈

## 3. 자신의 default branch에서 닫힌다는 점을 주의하자!!


# Reference
> [[Git] 좋은 커밋 메세지 작성하기위한 규칙들](https://beomseok95.tistory.com/328)   
> [좋은 git 커밋 메시지를 작성하기 위한 7가지 약속](https://meetup.toast.com/posts/106)   
> [Linking a pull request to an issue - Github Docs](https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue)
