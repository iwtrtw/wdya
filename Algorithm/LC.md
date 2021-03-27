```java
int[] arr = new int[n]
Arrays.srot(arr)
```

```
Math.abs(-1)
```

```java
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
int T =Integer.parseInt(br.readLine().trim());
char[] seat = br.readLine().trim().toCharArray();
ArrayList<PriorityQueue<Integer>> queueList = new ArrayList<>();
queueList.add(new PriorityQueue<Integer>());
queueList.add(new PriorityQueue<Integer>());
queueList.get(seat[i]-'0').offer(i+1);
```



#### 链表

##### 双指针

+ 倒数第k

```java
//left=k  right=null
class Solution {
    public ListNode getKthFromEnd(ListNode head, int k) {
        //head为null || k=0
        if(head ==null|| k==0){
            return null;
        }
        
        ListNode left = head, right = head;
        for(int i = 0; i < k; i++){
            //k>list.length
            if(right==null&&i<k){
                return null;
            }
            right = right.next;
        }
        while(right != null) {
            right = right.next;
            left = left.next;
        }
        return left;
    }
}
```

```java
        ListNode pre = head;
        ListNode curr = head;
        while(curr!=null){
            curr = curr.next;
            if(k>0)
                k--;
            else{
                pre = pre.next;
            }
        }
        return pre;
```

+ 获取中间位置的元素

```java
        //head为null
        if(head ==null){
            return null;
        }
        ListNode slow = head, fast = head;
		//n 为偶数时慢指针指向靠后结点
        while(fast != null && fast.next!=null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        return slow;
```

```java
        //head为null
        if(head ==null){
            return null;
        }
        ListNode slow = head, fast = head;
		//n 为偶数时慢指针指向靠前结点
        while(fast.next != null && fast.next.next!=null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        return slow;
```

+ 判断链表是否存在环

```java
        ListNode slow = head;
        ListNode fast = head;
        while(fast != null && fast.next!=null) {
            fast = fast.next.next;
            if(fast == slow) {
                return true;
            }
            slow = slow.next;
        }
        return fase;
```

+ 判断环的长度

      ListNode slow = head;
      ListNode fast = head;
      while(fast != null && fast.next!=null) {
          fast = fast.next.next;
          if(fast == slow) {
              return true;
          }
          slow = slow.next;
      }
      return fase;

