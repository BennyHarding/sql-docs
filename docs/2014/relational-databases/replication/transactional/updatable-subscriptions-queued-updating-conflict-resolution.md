---
title: "Queued Updating Conflict Detection and Resolution | Microsoft Docs"
ms.custom: ""
ms.date: "06/13/2017"
ms.prod: "sql-server-2014"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "replication"
ms.tgt_pltfrm: ""
ms.topic: conceptual
helpviewer_keywords: 
  - "conflict resolution [SQL Server replication], queued updating subscriptions"
  - "viewing queued updating conflicts"
  - "conflict resolution [SQL Server replication]"
  - "queued updating subscriptions [SQL Server replication]"
  - "articles [SQL Server replication], conflict resolution"
ms.assetid: 084ac587-25e7-4bd0-a385-556bbe07d02f
caps.latest.revision: 38
author: MashaMSFT
ms.author: mathoma
manager: craigg
---
# Queued Updating Conflict Detection and Resolution
  Because queued updating subscriptions allow modifications to the same data at multiple locations, there may be conflicts when data is synchronized at the Publisher. Replication detects any conflicts when changes are synchronized with the Publisher and resolves those conflicts using the resolution policy you selected when creating the publication. The following conflicts can occur:  
  
-   Update and insert conflicts. This conflict happens when the same data is changed at two locations. One change wins, and the other one loses.  
  
-   Delete conflicts. This conflict occurs when the same row is deleted at one location and changed at the other.  
  
 Conflict detection and resolution can be a time-consuming and resource-intensive process; therefore, it is best to minimize conflicts in the application by creating data partitions so that different Subscribers modify different subsets of data.  
  
## Detecting Conflicts  
 When creating a publication and enabling queued updating, replication adds a **uniqueidentifier** column (**msrepl_tran_version**) with the default of **newid()** to the underlying table. When published data is changed at either the Publisher or the Subscriber, the row receives a new globally unique identifier (GUID) to indicate that a new row version exists. The Queue Reader Agent uses this column during synchronization to determine if a conflict exists.  
  
 A transaction in a queue maintains the old and new row version values. When the transaction is applied at the Publisher, the GUIDs from the transaction and the GUID in the publication are compared. If the old GUID stored in the transaction matches the GUID in the publication, the publication is updated and the row is assigned the new GUID that was generated by the Subscriber. By updating the publication with the GUID from the transaction, you have matching row versions in the publication and in the transaction.  
  
 If the old GUID stored in the transaction does not match the GUID in the publication, a conflict is detected. The new GUID in the publication indicates that two different row versions exist: one in the transaction being submitted by the Subscriber and a newer one that exists on the Publisher. In this case, another Subscriber or the Publisher updated the same row in the publication before this Subscriber transaction was synchronized.  
  
 Unlike merge replication, the use of a GUID column is not used to identify the row itself, but is used to check if the row has changed.  
  
## Resolving Conflicts  
 When you create a publication using queued updating, you select a conflict resolver to be used if any conflicts are detected. The conflict resolver governs how the Queue Reader Agent handles different versions of the same row encountered during synchronization. You can change the conflict resolution policy after the publication is created as long as there are no subscriptions to the publication. The conflict resolver choices are the following:  
  
-   Publisher wins (the default)  
  
-   Publisher wins and the subscription is reinitialized  
  
-   Subscriber wins  
  
 Conflicts are recorded and can be viewed using the Conflict Viewer.  
  
 **To set the queued updating conflict resolution policy**  
  
-   [!INCLUDE[ssManStudioFull](../../../includes/ssmanstudiofull-md.md)]: [Set Queued Updating Conflict Resolution Options &#40;SQL Server Management Studio&#41;](../publish/set-queued-updating-conflict-resolution-options-sql-server-management-studio.md)  
  
-   Replication Transact-SQL programming: [Enable Updating Subscriptions for Transactional Publications](../publish/enable-updating-subscriptions-for-transactional-publications.md)  
  
 **To view data conflicts**  
  
-   [!INCLUDE[ssManStudioFull](../../../includes/ssmanstudiofull-md.md)]: [View Data Conflicts for Transactional Publications &#40;SQL Server Management Studio&#41;](../view-data-conflicts-for-transactional-publications-sql-server-management-studio.md)  
  
### Publisher Wins  
 When the conflict resolution is set to the Publisher wins, transactional consistency is maintained based on the data at the Publisher. The conflicting transaction is rolled back at the Subscriber that initiated it.  
  
 The Queue Reader Agent detects a conflict and compensating commands are generated and propagated to the Subscriber by posting them in the distribution database. The Distribution Agent then applies the compensating commands to the Subscriber that originated the conflicting transaction. The compensating actions update the rows on the Subscriber to match the rows on the Publisher.  
  
 Until the compensating commands are applied, it is possible to read the results of a transaction that will eventually be rolled back at the Subscriber. This is equivalent to a dirty read (read uncommitted isolation level). There is no compensation for the subsequent dependent transactions that can occur. However, transaction boundaries are honored and all the actions within a transaction are either committed, or in the case of a conflict, rolled back.  
  
### Publisher Wins and the Subscription Is Reinitialized  
 Reinitializing the Subscriber to resolve conflicts maintains strict transactional consistency at the Subscriber, but it can be time consuming if the publication contains large amounts of data.  
  
 When the Queue Reader Agent detects a conflict, all remaining transactions in the queue (including the transaction in conflict) are rejected, and the Subscriber is marked for reinitialization. The next snapshot generated for the publication is applied by the Distribution Agent to the Subscriber.  
  
### Subscriber Wins  
 Conflict detection under the Subscriber wins policy means the last Subscriber transaction to update the Publisher wins. In this case, when a conflict is detected, the transaction sent by the Subscriber is still used and the Publisher is updated. This policy is suitable for applications where such changes do not compromise data integrity.  
  
## See Also  
 [Updatable Subscriptions for Transactional Replication](updatable-subscriptions-for-transactional-replication.md)  
  
  