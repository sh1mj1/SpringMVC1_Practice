# SpringMVC1_Practice

# 1. 프로젝트 준비

프로젝트 준비

[https://start.spring.io/](https://start.spring.io/)


Lombok 설정

실행 속도 높이기

테스트 [http://localhost:8080](http://localhost:8080) 호출해서 Whitelabel Error Page 확인

### **Welcome 페이지 추가**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<ul>
    <li>상품관리
        <ul>
            <li><a href="/basic/items"> 상품관리 - 기본</a></li>
        </ul>
    </li>
</ul>

</body>
</html>
```


# 2. 요구사항 분석

상품을 관리할 수 있는 서비스 만들어 봅시다.

**상품 도메인 모델**

- 상품 ID
- 상품명
- 가격
- 수량

**상품 관리 기능**

- 상품 목록
- 상품 상세
- 상품 등록
- 상품 수정

**서비스 화면**

최상단 링크에서 확인

**서비스 제공 흐름**


> React, Vue, js 같은 웹 클라이언트 기술을 사용하고 웹 프론트엔드 개발자가 웹 퍼블리셔 역할까지 포함하는 경우도 있음.
이 때 프론트엔드 개발자가 HTML 을 동적으로 만들고 웹 화면의 흐름을 담당. 이 경우 벡엔드 개발자는 HTML 뷰 템플릿을 직접 만지는 대신, HTTP API 을 통해 클라이언트가 필요로하는 데이터, 기능을 제공하면 됨.

# 3. 상품 도메인 개발


**Item - 상품 도메인 클래스.**

```java
package hello.itemservice.domain.item;

import lombok.Data;

@Data
public class Item {

    private Long id;
    private String itemName;
    private Integer price;
    private Integer quantity;

    public Item(){}

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```

@Data 로 @Getter @Setter @RequiredArgsConstructor @ToString @EqualsAndHashCode. 을 만들어줌.

**`ItemRepository` - 상품 저장소**

```java
package hello.itemservice.domain.item;

@Repository
public class ItemRepository {
    private static final Map<Long, Item> store = new HashMap<>();
    private static long sequence = 0L;

    public Item save(Item item) {
        item.setId(++sequence);
        store.put(item.getId(), item);
        return item;
    }
    public Item findById(Long id) {
        return store.get(id);
    }
    public List<Item> findAll() {
        return new ArrayList<>(store.values());
    }
    public void update(Long itemId, Item updateParam) {
        Item findItem = findById(itemId);
        findItem.setItemName(updateParam.getItemName());
				findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }
    public void clearStore() {
        store.clear();
    }
}
```

Item 을 save 했을 때 Id 는 하나씩 증가하도록 설정함. 

**`ItemRepositoryTest` - 상품 저장소 테스트**

```java
package hello.itemservice.domain.item;

import static org.assertj.core.api.Assertions.assertThat;

class ItemRepositoryTest {

    ItemRepository itemRepository = new ItemRepository();

    @AfterEach
    void tearDown() {
        itemRepository.clearStore();
    }

    @Test
    void save() {
        // given
        Item item = new Item("itemA", 1000, 10);
        // when
        Item savedItem = itemRepository.save(item);
        // then
        Item findItem = itemRepository.findById(item.getId());
        assertThat(findItem).isEqualTo(savedItem);
    }

    @Test
    void findAll() {
        // given
        Item item1 = new Item("item1", 1000, 10);
        Item item2 = new Item("item2", 2000, 20);
        itemRepository.save(item1);
        itemRepository.save(item2);
        // when
        List<Item> result = itemRepository.findAll();
        // then
        assertThat(result.size()).isEqualTo(2);
        assertThat(result).contains(item1, item2);
    }

    @Test
    void updateItem() {
        // given
        Item item = new Item("item1", 1000, 10);
        Item savedItem = itemRepository.save(item);
        Long itemId = savedItem.getId();
        // when
        Item updateParam = new Item("item2", 2000, 30);
        itemRepository.updateItem(itemId, updateParam);
        Item findItem = itemRepository.findById(itemId);
        // then
        assertThat(findItem.getItemName()).isEqualTo(updateParam.getItemName());
        assertThat(findItem.getPrice()).isEqualTo(updateParam.getPrice());
        assertThat(findItem.getQuantity()).isEqualTo(updateParam.getQuantity());

    }
}
```

실제 서비스를 구축할 때는 HashMap 과 Long 대신 동시성을 고려하여 ConcurrentHashMap 과 AtomicLong 을 사용해야 한다.

update() 의 경우 Item 객체의 id 를 사용하지 않으므로 ItemUpdateDto 와 같이 id 를 포함하지 않는 객체를 만드는 것이 명확하다. 중복을 피하는 것과 명확성을 높이는 것 중 명확성을 높이는 것이 좋음.


# 4. 상품 서비스 HTML

핵심 비즈니스 로직을 개발하는 동안, 웹 퍼블리셔는 HTML 마크업을 완료했다.다음 파일들을 경로에 넣고 잘 동작하는지 확인해보자.

**부트스트랩**

참고로 HTML을 편리하게 개발하기 위해 부트스트랩을 사용했다. 먼저 필요한 부트스트랩 파일을 설치하자.

- **부트스트랩 공식 사이트:** [https://getbootstrap.com](https://getbootstrap.com/)
- **부트스트랩을 다운로드 받고 압축을 풀자.**
    - 이동: [https://getbootstrap.com/docs/5.0/getting-started/download/](https://getbootstrap.com/docs/5.0/getting-started/download/)
    - Compiled CSS and JS 항목을 다운로드하자.
    - 압축을 풀고 bootstrap.min.css를 복사해서 다음 폴더에 추가하자
    - resources/static/css/bootstrap.min.css

**참고**부트스트랩(Bootstrap)은 웹사이트를 쉽게 만들 수 있게 도와주는 HTML, CSS, JS 프레임워크이다. 하나의 CSS로 휴대폰, 태블릿, 데스크탑까지 다양한 기기에서 작동한다. 다양한 기능을 제공하여 사용자가 쉽게 웹사이트를 제작, 유지, 보수할 수 있도록 도와준다.

**HTML, css 파일**

- /resources/static/css/bootstrap.min.css **→** 부트스트랩 다운로드
- /resources/static/html/`items.html`
    
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <link href="../css/bootstrap.min.css" rel="stylesheet">
        <title>Title</title>
    </head>
    <body>
    <div class="container" style="max-width: 600px">
        <div class="py-5 text-center">
            <h2> 상품 목록</h2>
        </div>
        <div class="row">
            <div class="col">
                <button class="btn btn-primary float-end"
                        onclick="location.href='addForm.html'" type="button">상품 등록
                </button>
            </div>
        </div>
    
        <hr class="my-4">
        <div>
            <table class="table">
                <thead>
                <tr>
                    <th>ID</th>
                    <th>상품명</th>
                    <th>가격</th>
                    <th>수량</th>
                </tr>
                </thead>
                <tbody>
                <tr>
                    <td><a href="items.html">2</a></td>
                    <td><a href="items.html">테스트 상품2</a></td>
                    <td>20000</td>
                    <td>20</td>
                </tr>
                </tbody>
            </table>
        </div>
    </div>
    
    </body>
    </html>
    ```
    
- /resources/static/html/`item.html`
    
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <link href="../css/bootstrap.min.css" rel="stylesheet">
        <style>
            .container {
                max-width: 560px;
            }
        </style>
    </head>
    <body>
    <div class="container">
        <div class="py-5 text-center">
            <h2>상품 상세</h2>
        </div>
        <div>
            <label for="itemId">상품 ID</label>
            <input type="text" id="itemId" name="itemId"
                   class="form-control" value="1" readonly>
        </div>
        <div>
            <label for="itemName">상품명</label>
            <input type="text" id="itemName" name="itemName"
                   class="form-control" value="상품A" readonly>
        </div>
        <div>
            <label for="price">가격</label>
            <input type="text" id="price" name="price"
                   class="form-control" value="10000" readonly>
        </div>
        <div>
            <label for="price">가격</label>
            <input type="text" id="quantity" name="quantity"
                   class="form-control" value="10" readonly>
        </div>
        <hr class="my-4">
        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg"
                        onclick="location.href='editForm.html'" type="button">상품 수정</button>
            </div>
            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg"
                        onclick="location.href='items.html'" type="button">목록으로
                </button>
            </div>
        </div>
        <!--container-->
    </div>
    </body>
    </html>
    ```
    
- /resources/static/html/`addForm.html`
    
    ```html
    <!DOCTYPE html>
    <html lang="en" xmlns="http://www.w3.org/1999/html">
    <head>
        <meta charset="UTF-8">
        <link rel="stylesheet" href="../css/bootstrap.min.css">
        <title>Title</title>
        <style>
            .container {
                max-width: 560px;
            }
        </style>
    </head>
    <body>
    <div class="container">
        <div class="py-5 text-center">
            <h2>상품 등록 폼</h2>
        </div>
        <h4 class="mb-3">상품 입력</h4>
        <form action="item.html" method="post">
            <div>
                <label for="itemName">상품명</label>
                <input type="text" id="itemName" name="itemName"
                       class="form-control" placeholder="이름을 입력하세요">
            </div>
            <div>
                <label for="price">가격</label>
                <input type="text" id="price" name="price"
                       class="form-control" placeholder="가격을 입력하세요">
            </div>
            <div>
                <label for="quantity">수량</label>
                <input type="text" id="quantity" name="quantity"
                       class="form-control" placeholder="수량을 입력하세요">
            </div>
            <hr class="my-4">
            <div class="row">
                <div class="col">
                    <button class="w-100 btn btn-primary btn-lg">상품 등록</button>
                </div>
                <div class="col">
                    <button class="w-100 btn btn-primary btn-lg"
                            onclick="location.href='items.html'"
                            type="button">취소
                    </button>
                </div>
            </div>
        </form>
    
    </div>
    
    </body>
    </html>
    ```
    
- /resources/static/html/`editForm.html`
    
    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <link rel="stylesheet" href="../css/bootstrap.min.css">
        <style>
            .container {
                max-width: 560px;
            }
        </style>
    </head>
    <body>
    <div class="container">
        <div class="py-5text-center">
            <h2>상품 수정 폼</h2>
        </div>
        <form action="item.html" method="post">
            <div>
                <label for="id">상품 ID</label>
                <input type="text" id="id"
                       name="id" class="form-control"
                       value="1" readonly>
            </div>
    
            <div>
                <label for="itemName">상품명</label>
                <input type="text" id="itemName"
                       name="itemName" class="form-control" value="상품A">
            </div>
    
            <div>
                <label for="price">가격</label>
                <input type="text" id="price"
                       name="price" class="form-control" value="10000">
            </div>
    
            <div>
                <label for="quantity">수량</label>
                <input type="text" id="quantity"
                       name="quantity" class="form-control" value="10">
            </div>
    
            <hr class="my-4">
            <div class="row">
                <div class="col">
                    <button class="w-100 btn btn-primary btn-lg" type="submit">
                        저장
                    </button>
                </div>
                <div class="col">
                    <button class="w-100 btn btn-secondary btn-lg"
                            onclick="location.href='item.html'" type="button">취소
                    </button>
    
                </div>
            </div>
    
        </form>
    </div>
    
    </body>
    </html>
    ```
    

이렇게 정적 리소스가 공개되는 /resources/static 폴더에 HTML 을 넣어두면, 실제 서비스에서도 공개됨. 만약 서비스를 운영한다면 지금처럼 공개할 필요 없는 HTML 을 두는 것은 주의합시다.