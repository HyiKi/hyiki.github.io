---
title: Propagation-Types 事务传播特性
date:  2022-09-05 12:39:28 +0800
category: Reference
tags: Propagation types
excerpt: 介绍八种事务传播特性
---

* content
{:toc}

甚麼是Propagation(傳播)呢？當一個Transaction方法(method)碰到另一個Transaction方法(method)時的處理行為。

Propagation Types(傳播類型)一共有八種，說明如下:

| **序號** | **Propagation Types & Description**傳播類型與說明           |
| -------- | ------------------------------------------------------------ |
| 1        | **TransactionDefinition.PROPAGATION_MANDATORY**<br />◆說明-- Supports a current transaction; throws an exception if no current transaction exists. (支持當前 Transaction；如果當前 Transaction 不存在，則引發異常。)<br />◆Spring org.springframework.transaction.annotation.Propagation.MANDATORY |
| 2        | **TransactionDefinition.PROPAGATION_NESTED**<br />◆說明-- Executes within a nested transaction if a current transaction exists. (如果當前 Transaction 存在，則在嵌套 Transaction 中執行。)<br />◆Spring org.springframework.transaction.annotation.Propagation.NESTED |
| 3        | **TransactionDefinition.PROPAGATION_NEVER**<br />◆說明-- Does not support a current transaction; throws an exception if a current transaction exsts. (不支持當前 Transaction；如果當前 Transaction 存在，則引發異常。)<br />◆Spring org.springframework.transaction.annotation.Propagation.NEVER |
| 4        | **TransactionDefinition.PROPAGATION_NOT_SUPPORTED**<br />◆說明-- Does not support a current transaction; rather always execute nontransactionally. (不支持當前 Transaction；而是始終以非 Transaction 方式執行。)<br />◆Spring org.springframework.transaction.annotation.Propagation.NOT_SUPPORTED |
| 5(常用)  | **TransactionDefinition.PROPAGATION_REQUIRED**<br />◆說明-- Supports a current transaction; creates a new one if none exists. (支持當前 Transaction；如果不存在，則創建一個新的。)<br />◆Spring org.springframework.transaction.annotation.Propagation.REQUIRED**<br />◆**This is the **default** **setting of a transaction annotation. (Spring @Transaction Propagation**屬性之預設值) |
| 6(常用)  | **TransactionDefinition.PROPAGATION_REQUIRES_NEW**<br />◆說明-- Creates a new transaction, suspending the current transaction if one exists. (創建一個新 Transaction，如果 Transaction 存在則暫停當前 Transaction 。)<br />◆Spring org.springframework.transaction.annotation.Propagation.REQUIRES_NEW |
| 7        | **TransactionDefinition.PROPAGATION_SUPPORTS**<br />◆說明-- Supports a current transaction; executes non-transactionally if none exists. (支持當前 Transaction；如果不存在，則以非 Transaction 方式執行。)<br />◆Spring org.springframework.transaction.annotation.Propagation.SUPPORTS |
| 8        | **TransactionDefinition.TIMEOUT_DEFAULT**<br />◆說明-- Uses the default timeout of the underlying transaction system, or none if timeouts are not supported. (使用基礎 Transaction 系統的默認超時；如果不支持超時，則不使用默認超時。)<br />◆**Spring** **不支援** |

1. TransactionDefinition.PROPAGATION_MANDATORY （MANDATORY强制性）
   * 不开事务就报错
   * 开了用当前事务
   * 只要外层或内层报错无捕获就全部回滚，因为同一事务
2. TransactionDefinition.PROPAGATION_NESTED（NESTED嵌套）
   * 无论调用者是否开启事务，NESTED被调用者都会新起事务
   * 嵌套事务在外层commit
   * 外层异常影响内层
   * 内层异常不影响外层
3. TransactionDefinition.PROPAGATION_NEVER（NEVER从不）
   * 调用者开了事务就报错
4. TransactionDefinition.PROPAGATION_NOT_SUPPORTED（NOT_SUPPORTED不支持）
   * NOT_SUPPORTED被调用者始终不开启/不支持事务
5. 【default、常用】**TransactionDefinition.PROPAGATION_REQUIRED**（REQUIRED需要）
   * 支持调用者开启的事务
   * 如果调用者未开启事务，则在本方法创建一个新事务
   * 只要外层或内层报错无捕获就全部回滚，因为同一事务
6. 【常用】**TransactionDefinition.PROPAGATION_REQUIRES_NEW**
   * 无论调用者是否开启事务，都会创建一个新事务
   * 事务相互独立
7. TransactionDefinition.PROPAGATION_SUPPORTS（SUPPORTS支持）
   * **※在此傳播屬性下，被調用者是否有 Transaction，完全依賴於調用者，調用者有 Transaction 則有 Transaction，調用者沒 Transaction 則沒 Transaction 。**
   * 在开启事务的情况下，只要外层或内层报错无捕获就全部回滚，因为同一事务
