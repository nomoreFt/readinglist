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

