```java
class LRUCache {

    class ListNode {
        int key, val;
        ListNode pre, next;

        public ListNode(int key, int val) {
            this.key = key;
            this.val = val;
        }

        public ListNode() {}
    }

    int capacity, content;
    ListNode head, tail;
    Map<Integer, ListNode> map;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.content = 0;
        map = new HashMap<>();
        head = new ListNode();
        tail = new ListNode();
        head.next = tail;
        tail.pre = head;
    }
    
    public int get(int key) {
        ListNode node = map.get(key);
        if (node == null)
            return -1;
        else {
            moveToHead(node);
            return node.val;
        }
    }
    
    public void put(int key, int value) {
        ListNode node = map.get(key);
        if (node == null) {
            ListNode newNode = new ListNode(key, value);
            map.put(key, newNode);
            addToHead(newNode);
            ++ content;
            if (content > capacity) {
                ListNode deleteNode = removeTail();
                map.remove(deleteNode.key);
                -- content;
            }
        } else {
            node.val = value;
            moveToHead(node);
        }
    }

    private void moveToHead(ListNode node) {
        removeNode(node);
        addToHead(node);
    }

    private void addToHead(ListNode node) {
        head.next.pre = node;
        node.next = head.next;
        head.next = node;
        node.pre = head;
    }

    private ListNode removeTail() {
        ListNode node = tail.pre;
        removeNode(node);
        return node;
    }

    private void removeNode(ListNode node) {
        node.pre.next = node.next;
        node.next.pre = node.pre;
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```