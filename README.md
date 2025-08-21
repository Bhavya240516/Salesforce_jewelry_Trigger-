# Salesforce_jewelry_Trigger-

trigger OrderTotalTrigger on Jewel_Order_Item__c (after insert, after update, after delete, after undelete) {
    
    // Step 1: Get the parent Order IDs from the trigger records.
    Set<Id> orderIds = new Set<Id>();
    if (Trigger.isDelete) {
        for (Jewel_Order_Item__c oldItem : Trigger.old) {
            orderIds.add(oldItem.Order__c);
        }
    } else {
        for (Jewel_Order_Item__c newItem : Trigger.new) {
            orderIds.add(newItem.Order__c);
        }
    }
    
    // Step 2: Use a Map to store the new calculated total for each order.
    Map<Id, Decimal> orderTotalMap = new Map<Id, Decimal>();
    
    // Step 3: Query all related Jewel_Order_Item__c records for the orders.
    List<Jewel_Order_Item__c> orderItems = [SELECT Order__c, Subtotal__c FROM Jewel_Order_Item__c WHERE Order__c IN :orderIds];
    
    // Step 4: Loop through the items and calculate the total for each order.
    for (Jewel_Order_Item__c item : orderItems) {
        if (orderTotalMap.containsKey(item.Order__c)) {
            orderTotalMap.put(item.Order__c, orderTotalMap.get(item.Order__c) + item.Subtotal__c);
        } else {
            orderTotalMap.put(item.Order__c, item.Subtotal__c);
        }
    }
    
    // Step 5: Prepare the Order records for the update.
    List<Order__c> ordersToUpdate = new List<Order__c>();
    for (Id orderId : orderTotalMap.keySet()) {
        Order__c order = new Order__c(Id = orderId);
        order.Total_Amount__c = orderTotalMap.get(orderId);
        ordersToUpdate.add(order);
    }
    
    // Step 6: Update the Order records.
    if (!ordersToUpdate.isEmpty()) {
        update ordersToUpdate;
    }
}
