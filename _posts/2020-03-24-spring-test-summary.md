---
layout: post
title:  "[Spring Testing] Ghi chú test"
date:   2019-03-23 15:23:00 +0700
categories: java
---

# Test controller
**Khai báo test controller**
```java
@ExtendWith(SpringExtension.class)
@WebMvcTest(AccountController.class)
class AccountControllerTests {

    @MockBean
    private AccountService accountService;

    @Autowired
    private AccountController controller;

    @Autowired
    private MockMvc mockMvc;
}
```
## Test method GET/DELETE
```java
@Test
    void testGet() throws Exception {
    
       given(service.testMethod()).willReturn();
       mockMvc.perform(MockMvcRequestBuilders
           .get("/accounts"))
           .andDo(rs -> System.out.println(rs.getResponse().getContentAsString()))
           .andExpect(status().isOk());

        doNothing().when(accountService).delete(1);
        mockMvc.perform(MockMvcRequestBuilders
            .delete("/accounts/1"))
            .andExpect(status().isNoContent());
//        JSONObject
//        mockMvc.perform(MockMvcRequestBuilders
//            .post("/accounts")
//            .contentType(MediaType.APPLICATION_JSON)
//        .content())
//            .andDo(rs -> System.out.println(rs.getResponse().getContentAsString()))
//            .andExpect(status().isOk());
    }

```