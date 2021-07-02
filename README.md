# 본격 읽은 책 리스트 페이지 만들기

- 도메인 정의

    app의 중심이 되는 도메인 개념은 독자의 독서 목록 책이다.

    ```java
    @Entity
    @Getter
    @Setter
    @NoArgsConstructor
    public class Book {
    	
    	@Id
    	@GeneratedValue(strategy=GenerationType.AUTO)
    	private Long id;
    	private String reader;
    	private String isbn;
    	private String title;
    	private String author;
    	private String description;
    }
    ```

    - @Entity : 클래스를 JPA 엔티티로 지정
    - @Id , @GeneratedValue(strategy=GenerationType.AUTO) : id 필드에 엔티티의 유일성을 식별, 자동으로 값을 제공하는 필드로 지정
    - 롬복을 이용하여 자동으로 게터세터,생성자 생성되게 지정

- 레포지토리 인터페이스 선언

    Book 객체를 영속화할 수 있는  레퍼지토리 선언. 

    ```java
    package readinglist;

    import org.springframework.data.jpa.repository.JpaRepository;

    import java.util.List;

    public interface ReadingListRepository extends JpaRepository<Book, Long> {
        List<Book> findByReader(String reader);
    }
    ```

    JpaRepository를 확장하여 메서드 18개 상속받는다. 매개변수로는 레퍼지토리가 사용할 도메인 타입(Book), 하나는 ID (@Id Long id) 인터페이스 상속받는 메서드는 구현할 필요 없다. 런타임시에 자동으로 구현된다.

- 웹 인터페이스 만들기

    **컨트롤러 만들기**

    ```java
    package readinglist;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    import java.util.List;

    @Controller
    @RequestMapping("/")
    public class ReadingListController {
        private static final String reader = "craig";

        private ReadingListRepository readingListRepository;

        @Autowired
        public ReadingListController(ReadingListRepository readingListRepository){
            this.readingListRepository = readingListRepository;
        }

        @RequestMapping(method = RequestMethod.GET)
        public String readersBooks(Model model){
            List<Book> readingList = readingListRepository.findByReader(reader);
            if(readingList != null){
                model.addAttribute("books",readingList);
            }
            return "readingList";
        }

        @RequestMapping(method = RequestMethod.POST)
        public String addReadingList(Book book){
            book.setReader(reader);
            readingListRepository.save(book);
            return "redirect:/";
        }
    }
    ```

    컴포넌트 검색으로 자동으로 스프링 app 컨텍스트에 빈으로 등록하게 @Controller 붙임. 요청을 처리하는 모든 메서드를 기본 URL 경로인 "/"로 처리하게 @RequestMapping 어노테이션을 붙였다.

    - 메서드 1 readersBook () : 생성자로 주입된 레포지토리에서 reader로 목록 조회하는 메서드이다. /로 들어오는 HTTP GET 요청을  처리하여 "readingList"라는 뷰를 반환한다.
    - 메서드 2 addToReadingList() : 요청 바디에 있는 Data를 Book 객체에 바인드하여 /로 들어오는 HTTP POST 요청을 처리한다. /로 리다이렉트 하도록 명시하며 반환한다. (이 리다이렉트 경로로 들어오는 요청은 다른 컨트롤러 메서드가 처리한다.)



### html,css

Thymeleaf html과 선언된 css 제작

```html
<!DOCTYPE html>
<html>
<head>
  <title>Reading List</title>
  <link rel="stylesheet" th:href="@{/css/style.css}"></link>
</head>
<body>
  <h2>Your Reading List</h2>
  <div th:unless="${#lists.isEmpty(books)}">
    <dl th:each="book : ${books}">
      <dt class="bookHeadline">
        <span th:text="${book.title}">Title</span> by
        <span th:text="${book.author}">Author</span>
        (ISBN: <span th:text="${book.isbn}">ISBN</span>)
      </dt>
      <dd class="bookDescription">
        <span th:if="${book.description}"
              th:text="${book.description}">Description</span>
        <span th:if="${book.description eq null}">
              No description available</span>
      </dd> 
    </dl>
  </div>
  <div th:if="${#lists.isEmpty(books)}">
    <p>You have no books in your book list</p>
  </div>
  <hr/>
    
  <h3>Add a book</h3>
  <form method="POST" th:action="@{/}">
    <label for="title">Title:</label>
    <input type="text" name="title" size="50"></input><br />
    <label for="author">Author:</label>
    <input type="text" name="author" size="50"></input><br />
    <label for="isbn">ISBN:</label>
    <input type="text" name="isbn" size="15"></input><br />
    <label for="description">Description:</label><br />
    <textarea name="description" cols="80" rows="5"></textarea><br />
    <input type="submit" value="Add Book" />
  </form>
</body>
</html>
```

```css
body {
    background-color: #cccccc;
    font-family: arial, helvetica, sans-serif;
}

.bookHeadline {
    font-size: 12pt;
    font-weight: bold;
}

.bookDescription {
    font-size: 10pt;
}

label {
    font-weight: bold;
}
```

## Thymeleaf

> **Thymeleaf에 대해**

Thymleaf는 View Template Engine이다. Controller에서 전달받은 데이터를 이용해 동적인 html 페이지를 만들 수 있다.

> Controller

```java
@RequestMapping(method = RequestMethod.GET)
    public String readersBooks(Model model){
        List<Book> readingList = readingListRepository.findByReader(reader);
        if(readingList != null){
            model.addAttribute("books",readingList);
        }
        return "readingList";
    }
```

thymeleaf의 default prefix는 src/main/resources/templates이며 suffix는 .html입니다.

**src/main/resources/templates/readingList.html** 인 뷰를 리턴하고 있습니다.

> html

```java
<div th:unless="${#lists.isEmpty(books)}">
    <dl th:each="book : ${books}">
        <dt class="bookHeadline">
            <span th:text="${book.title}">Title</span> by
            <span th:text="${book.author}">Author</span>
            (ISBN: <span th:text="${book.isbn}">ISBN</span>)
        </dt>
        <dd class="bookDescription">
        <span th:if="${book.description}"
              th:text="${book.description}">Description</span>
            <span th:if="${book.description eq null}">
              No description available</span>
        </dd>
    </dl>
</div>
```

> Thymeleaf 문법 정리

**1) ${...} 표현식**

${...} 표현식을 이용해 컨트롤러에서 전달받은 변수에 접근할 수 있으며 th:속성명 내에서만 사용할 수 있습니다.

**2) @{...} 표현식**

@{...} 표현식은 서버의 contextPath를 의미하며 @{/} 는 "/contextPath/" 를 의미합니다.

**3) 문자 합치기**

합치고 싶은 문자열을 "|" 으로 감싸거나 + 연산자를 이용해 문자를 합칠 수 있습니다.

```java
<div th:text="|My name is ${info.name} !! |"></div>
<div th:text="'My name is '+ ${info.name} + ' !! '"></div>
```

**4) 비교 연산자**

```java
<!-- 이항 연산자 -->
<div th:text="${info.name != 'kim'}"></div>
<div th:text="${info.age >= 30}"></div>

<!-- 삼항 연산자 -->
<div th:text="${info.name == 'kim' ? 'ok' : 'no'}"></div>
```

**5) th:text**

태그 내에 text를 수정합니다.

```java
<div th:text="${info.name}"></div>
 <!--<div>kim</div>-->
```

**6) th:value**

태그 내에 value를 수정합니다.

```java
<input type='text' th:value="${info.name}">
```

**7) th:if, th:unless**

if else 속성을 이용해 조건문을 표현합니다.

```java
<th:block th:if="${info.age < 30}">당신은 30대가 아닙니다.</th:block>
<th:block th:unless="${info.age >= 30}">당신은 30대입니다.</th:block>
```

**8) th:switch, th:case**

switch문을 이용해 조건문을 표현합니다.

```java
<th:block th:switch="${info.name}">
  <div th:case="lee"> 당신은 이씨 입니다.</div>
  <div th:case="kim"> 당신은 김씨 입니다.</div>
</th:block>
```

**9) th:each**

each문을 이용해 반복문을 표현합니다.

```java
<th:block th:each="data:${datas}">
	<h1 th:text="${data}"></h1>
</th:block>
```

변수명 앞에 status 변수를 추가해 row에 대한 추가정보를 얻을 수 있습니다.

```java
<th:block th:each="data,status:${datas}">
	<h1 th:text="|${status.count} ${data}|"></h1>
</th:block>
```

status 속성

index : 0부터 시작
count : 1부터 시작
size : 총 개수
current : 현재 index의 변수
event/odd : 짝수/홀수 여부
first/last : 처음/마지막 여부
