---
created: 2025-11-07T17:55
updated: 2026-01-25T13:42
작성 완료: 1
---
[url](https://wildeveloperetrain.tistory.com/282)
# 사용법
## 1:1 연관관계
```java
public class User{
...
@OneToOne
@JoinTable(name="컬럼명")
private UserInfo userInfo;
}
```

## 1:N 연관관계
```java
@Entity
public class Board {
    @Id
    @Column(name = "board_id")
    private Long id;

    @Column(name = "title")
    private String title;

    @OneToMany
    @JoinTable(name = "board_comment",
            joinColumns = @JoinColumn(name = "board_id"),
            inverseJoinColumns = @JoinColumn(name = "comment_id"))
    private List<Comment> comments = new ArrayList<Comment>();
    
    ...
}

```

``` java
@Entity
public class Comment {
    @Id
    @Column(name = "comment_id")
    private Long id;

    @Column(name = "contents")
    private String contents;
    
    ...
    // getter, setter
}
```

