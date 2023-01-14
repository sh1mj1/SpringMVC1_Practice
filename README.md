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

