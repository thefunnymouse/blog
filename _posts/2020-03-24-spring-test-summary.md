---
layout: post
title:  "[Spring Testing] Ghi chú test"
date:   2019-03-23 15:23:00 +0700
categories: java
---

{% include toc.html %}

---

## Code dùng trong ví dụ

[AccountController](#code-accountcontroller)

[AccountService](#code-accountservice)

## Cú pháp Mockito
**Hàm có giá trị trả về:**

`when(account).thenReturn(...)`

<br>

**Hàm _void_:**

`doNothing().when(object).method()`

_Lưu ý:_ method của object sẽ nằm bên ngoài when()

<br>

**Throw exception:**

`when(...).thenThrow(new <Tên lớp exception>())`

---
## Các hàm asserts
**Assert equals:**

`Assertions.assertEquals(<giá trị mong đợi>, object.method());`

_`object.method()` là lời gọi tới hàm muốn test_

<br>

**Assert true/false**

Dùng để test biểu thức boolean hoặc hàm trả về kiểu boolean, ví dụ hàm `existsById(id)` trả về account có tồn tại hay không:

`assertTrue(accountRepository.existsById(1));`

<br>

**Kiểm tra hàm có được gọi**

Dùng trong trường hợp hàm có kiểu void.

`Mockito.verify(object).method()`

_Áp dụng:_
```java
   @Test
    @DisplayName("Delete - Existed")
    public void testDelete() {
        int id = 1;
        when(accountRepository.existsById(id)).thenReturn(true);
        doNothing().when(accountRepository).deleteById(id);

        // gọi hàm cần test
        accountService.delete(id);

        // kiểm tra xem hàm deleteById có được gọi
        verify(accountRepository).deleteById(id);
    }
```

<br>

**Kiểm tra throw exception**
```java
// Lấy ra exception
DataNotFoundException exception = assertThrows(
    DataNotFoundException.class,
    () -> accountService.update(1, putAccount)
);

// Kiểm tra message của exception
String expectedMessage = "Could not find Account with [id=1]";
assertEquals(expectedMessage, exception.getMessage());
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

### Code AccountService
```java
@Service
public class AccountService {
    private AccountRepository accountRepository;
    private TransactionRepository transactionRepository;

    public AccountService(
        AccountRepository accountRepository,
        TransactionRepository transactionRepository
    ) {
        this.accountRepository = accountRepository;
        this.transactionRepository = transactionRepository;
    }

    public List<Account> find() {
        return accountRepository.findAll();
    }

    public Account create(PostAccount postAccount) {
        Account account = new Account(
            0,
            postAccount.getName(),
            postAccount.getInit(),
            0
        );
        return accountRepository.save(account);
    }

    public Account update(int id, PutAccount putAccount) {
        Account account = accountRepository
            .findById(id)
            .orElseThrow(() -> new DataNotFoundException("Could not find Account with [id=" + id + "]"));

        if (putAccount.getName() == null && putAccount.getInit() == null) {
            throw new InvalidDataException("Invalid update account data, make sure either [name] or [init] not null");
        }

        if (putAccount.getName() != null) {
            account.setName(putAccount.getName());
        }

        if (putAccount.getInit() != null) {
            // Độ lệch giữa số tiền muốn update lại
            int updateRange = account.getInit() - putAccount.getInit();
            int remain = account.getInit() - account.getSpend();
            System.out.println(updateRange);
            // Chỉ update khi remain >= 0
            boolean canUpdate = updateRange <= remain;
            if (canUpdate) {
                account.setInit(putAccount.getInit());
            }
        }

        return accountRepository.save(account);
    }

    public void delete(int id) {
        if (accountRepository.existsById(id)) {
            accountRepository.deleteById(id);
        }
    }

    @Transactional
    public void transfer(PutTransfer putTransfer) {
        int fromId = putTransfer.getFromId();
        int toId = putTransfer.getToId();
        int total = putTransfer.getTotal();

        Account fromAccount = accountRepository
            .findById(fromId)
            .orElseThrow(() ->
                new DataNotFoundException("Could not find Account with [id=" + fromId + "]")
            );
        Account toAccount = accountRepository
            .findById(toId)
            .orElseThrow(() ->
                new DataNotFoundException("Could not find Account with [id=" + toId + "]")
            );

        if (fromAccount.getInit() - fromAccount.getSpend() < total) {
            throw new InvalidDataException("Not enough money to transfer");
        }

        fromAccount.setInit(fromAccount.getInit() - total);
        toAccount.setInit(toAccount.getInit() + total);
        accountRepository.saveAll(Arrays.asList(fromAccount, toAccount));

        Transaction transaction = new Transaction(0, fromId, toId, total, Timestamp.from(Instant.now()));
        transactionRepository.save(transaction);
    }
}

```