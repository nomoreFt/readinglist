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
