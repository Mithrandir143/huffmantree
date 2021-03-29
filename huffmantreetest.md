```java
package huffmantree;

import java.util.ArrayList;
import java.util.Collections;

/**
 * @author Mithrandir
 * @date 2021-02-2021/2/25-15:12
 */
public class HuffmanTreeTest {
    public static void main(String[] args) {
        int[] arr = {13, 7, 8, 3, 29, 6, 1};
        Node huffTree = HuffmanTree.creatHuffTree(arr);
        huffTree.preOrder();
    }
}
class HuffmanTree{
    public static Node creatHuffTree(int[] arr){
        ArrayList<Node> list = new ArrayList<>();
        for(int tmp : arr){
            list.add(new Node(tmp));
        }

        while (list.size()>1){
            //对list按照node的num进行从小到大排序
            Collections.sort(list);
            //找出list里面num最小的两个值
            Node left = list.get(0);
            Node right = list.get(1);
            //将最小的两个值进行合并
            Node parent = new Node(left.num + right.num);
            //构建一棵新的二叉树
            parent.leftNode=left;
            parent.rightNode=right;
            //删除用过的两个值
            list.remove(left);
            list.remove(right);
            //将新创建得来的值加入list
            list.add(parent);
            
        }
        return list.get(0);
    }
}
class Node implements Comparable<Node>{
    int num;
    Node leftNode;
    Node rightNode;


    //写一个前序遍历
    public void preOrder(){
        System.out.println(this.num);
        if(this.leftNode!=null){
            this.leftNode.preOrder();
        }
        if(this.rightNode!=null){
            this.rightNode.preOrder();
        }
    }
    @Override
    public String toString() {
        return "Node{" +
                "num=" + num +
                '}';
    }

    public Node(int num) {
        this.num = num;
    }

    @Override
    public int compareTo(Node o) {
        return this.num-o.num;
    }
}
```

