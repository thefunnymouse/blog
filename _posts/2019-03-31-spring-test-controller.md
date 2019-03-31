---
layout: post
title:  "[Spring Testing] Các cú pháp test controller"
date:   2019-03-31 15:23:00 +0700
categories: java
---

{% include toc.html %}

---

## Code dùng trong ví dụ

[AccountController](#code-accountcontroller)


## Cách sử dụng MockMvc

Để test **RestController**, ta **không thể gọi trực tiếp hàm** trong controller đó, mà phải gọi **thông qua API**, việc gửi request sẽ dùng `MockMvc`

Và để test controller ta dùng `MockMvc`, `MockMvc` hỗ trợ thực thi các request, và thực hiện các kiểm tra kết quả trả về từ controller.

Ví dụ gửi 1 request:
```java
mockMvc.perform(MockMvcRequestBuilders.get("/accounts"))
  .andDo(rs -> System.out.println(rs.getResponse().getContentAsString()))
  .andExpect(status().isOk())
  .andExpect(content().json("[{\"id\":0,\"name\":\"Tien\",\"init\":1234,\"spend\":5678}]"));
```

Với phương thức post, request builder sẽ phức tạp hơn, nên tách riêng ra cho dễ nhìn:
```java
RequestBuilder requestBuilder = MockMvcRequestBuilders.get("/accounts");
mockMvc.perform(requestBuilder)
  .andDo(rs -> System.out.println(rs.getResponse().getContentAsString()))
  .andExpect(status().isOk())
  .andExpect(content().json("[{\"id\":0,\"name\":\"Tien\",\"init\":1234,\"spend\":5678}]"));
```

### MockMvcRequestBuilders là cái gì?
Đây là 1 class gồm những **phương thức static** hỗ trợ **giả lập các request** để `MockMvc` có thể gửi tới controller dự định test.

#### API get hoặc delete
Hai loại API này thường không có body, nên có thể viết như sau:

```java
// Request get
RequestBuilder requestBuilder = MockMvcRequestBuilders.get("/accounts");

// Request delete
RequestBuilder requestBuilder = MockMvcRequestBuilders.delete("/accounts");
```

#### Thêm header cho request
```java
RequestBuilder requestBuilder = MockMvcRequestBuilders
  .get("/accounts")
  .header("Accept", "application/json");
```

#### API post hoặc put
Hai loại api này cần body và chỉ ra **định dạng** của body đó, ví dụ như json hay form-url-encoded, ...

Chỉ ra định dạng của body: `contentType()`

Gắn body vào: `content()`
```java
RequestBuilder requestBuilder = MockMvcRequestBuilders
  .get("/accounts")
  .contentType(MediaType.APPLICATION_JSON)
  .content(body.toString());
```

---

### Code AccountController
```java
@RestController
@Validated
@RequestMapping("/accounts")
public class AccountController {
    private AccountService accountService;

    @Autowired
    public AccountController(AccountService accountService) {
        this.accountService = accountService;
    }

    @GetMapping
    List<Account> getList() {
        return accountService.find();
    }

    @PostMapping(value = "/")
    Account create(@Valid @RequestBody PostAccount postAccount) {
        return accountService.create(postAccount);
    }

    @PutMapping(value = "/{id}")
    Account update(
        @PathVariable int id,
        @Valid @RequestBody PutAccount putAccount
    ) {
        return accountService.update(id, putAccount);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    void delete(@PathVariable int id) {
        accountService.delete(id);
    }

    @PutMapping("/transfer")
    @ResponseStatus(HttpStatus.ACCEPTED)
    void transfer(@Valid @RequestBody PutTransfer putTransfer) {
        accountService.transfer(putTransfer);
    }
}

```
