```java
package huffmancode;

import org.junit.Test;
import sun.awt.OSInfo;

import java.io.*;
import java.util.*;

/**
 * @author Mithrandir
 * @date 2021-02-2021/2/26-10:36
 */
public class HuffmanCode {
    public static void main(String[] args) {
        //要压缩的内容
        String content = "i like like like java do you like a java";
        //将要压缩的内容转为byte[]
        byte[] contentBytes = content.getBytes();
//        System.out.println("原始数组："+content);
//        System.out.println("原始数组的byte[]："+Arrays.toString(contentBytes));
//        //对byte[]的数据进行统计
//        List<Node> nodeList = getNodeList(contentBytes);
//        System.out.println("对应的List<Node>："+nodeList);
//        //根据统计的内容做成一个二叉树
//        Node root = creatHuffmanTree(nodeList);
//        System.out.println("二叉树的前序遍历：");
//        root.preOrder();
//        //根据二叉树，得到每个字符对应的二进制编码
//        getHuffmanMap(root);
//        System.out.println("Huffman编码表：" + huffmanMap);
//        byte[] huffmanBytes = zip(contentBytes, huffmanMap);
//        System.out.println("处理过后的byte[]："+Arrays.toString(huffmanBytes));
        byte[] bytes = huffmanZip(contentBytes);
        System.out.println("压缩后："+Arrays.toString(bytes));
    }

    /**
     * 对文件解压功能的测试
     */
    @Test
    public void testDecode(){
        String src = "D:\\Coding\\Bigdata\\exer\\赫夫曼压缩解压测试\\20180610063144.zip";
        String dest = "D:\\Coding\\Bigdata\\exer\\赫夫曼压缩解压测试\\0123.jpeg";
//        String src = "D:\Coding\Bigdata\exer\赫夫曼压缩解压测试\\JDBC基础面试题总结.zip";
//        String dest = "D:\Coding\Bigdata\exer\赫夫曼压缩解压测试\\JDBC基础面试题总结1.docx";
        unZipFile(src,dest);
        System.out.println("解压成功");
    }
    /**
     * @Author: Mithrandir
     * @Time: 2021/2/28-20:54
     * @Description: 对压缩完毕的文件进行解压
     * @param srcFile 要解压的压缩文件
     * @param destFile 压缩文件要解压到哪个文件
     * @return void
     */
    public static void unZipFile(String srcFile,String destFile) {

        ObjectInputStream ois = null;
        InputStream is = null;
        OutputStream os =null;
        try {
            //创建一个文件输入流
            is = new FileInputStream(srcFile);
            //创建一个和文件输入流相关的对象输入流
            ois = new ObjectInputStream(is);
            //读取byte[]
            byte[] huffmanBytes = (byte[]) ois.readObject();
            //读取赫夫曼编码表
            Map<Byte, String> unzip = (Map<Byte, String>) ois.readObject();
            //对已经读取到的byte[]进行解码
            byte[] bytes = decode(unzip, huffmanBytes);
            //创建一个文件输出流
            os = new FileOutputStream(destFile);
            //进行数据写出到文件
            os.write(bytes);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } finally {
            if (os != null) {
                try {
                    os.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (ois != null) {
                try {
                    ois.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    /**
     * fos的write只能写出byte[]或者是int类型，对于Map<>类型的数据需要采用oos
     */
    //自己写的文件压缩函数
//    public static void zipFile2(String srcFile,String destFile) throws Exception {
//        //造流
//        FileInputStream fis = new FileInputStream(srcFile);
//        //创建一个过度byte[]
//        byte[] bytes = new byte[fis.available()];
//        //读取文件
//        fis.read(bytes);
//        //对读取好的byte[]进行压缩
//        byte[] zipBytes = huffmanZip(bytes);
//        //创建一个写出流
//        FileOutputStream fos = new FileOutputStream(destFile);
//        ObjectOutputStream oos = new ObjectOutputStream(fos);
//        //进行数据写出
//        fos.write(zipBytes);
//        oos.writeObject(huffmanMap);
//    }

    /**
     * @Author: Mithrandir
     * @Time: 2021/2/28-21:01
     * @Description:
     * @param map 解码所需要的解码表
     * @param bytes 待解码的byte[]
     * @return byte 解码后的byte[]
     */
    public static byte[] decode(Map<Byte,String>map,byte[] bytes){
        StringBuilder builder = new StringBuilder();
        for (int i = 0; i < bytes.length; i++) {
            byte b = bytes[i];
            //设置一个flag用于判断是否为最后一个字节
            boolean flag = (i==bytes.length-1);
            builder.append(byteToBitString(flag, b));
            //根据压缩的赫夫曼编码得到其解码表
        }
        Map<String, Byte> unzipHuffmanMap = new HashMap<>();
        for (Map.Entry<Byte,String> tmp : map.entrySet()){
            unzipHuffmanMap.put(tmp.getValue(),tmp.getKey());
        }
        //创建一个集合用于存放byte
        ArrayList<Byte> list = new ArrayList<>();
        for (int i = 0; i < builder.length(); ) {
            //设置一个判断，用于确定是否匹配到unzipmap里的键值
            boolean flag = true;
            //创建一个简单的计数器
            int count = 1;
            Byte b =null;
            while (flag){
                String substring = builder.substring(i, i + count);
                b = unzipHuffmanMap.get(substring);
                if(b == null){
                    ++count;
                }else{
                    break;
                }
            }
            //将找到的byte添加进list中
            list.add(b);
            //把i直接移动到i+count的位置
            i+=count;
        }
        byte[] bytes1 = new byte[list.size()];
        for (int i = 0; i < bytes1.length; i++) {
            bytes1[i] = list.get(i);
        }
        return bytes1;
    }

    /**
     * @Author: Mithrandir
     * @Time: 2021/2/28-21:07
     * @Description: 将byte转换为string
     * @param flag 用于判断是否需要进行补位：如果是最后一个字节(flag为true)，则不需要进行补位；反之则反
     * @param b 要处理的字节
     * @return void
     */
    public static String byteToBitString(boolean flag,byte b){
        //将byte转换为int
        int tmp = b;
        if(!flag){//如果不是最后一个字节，则需要进行补位
            tmp |= 256;
        }
        String string = Integer.toBinaryString(tmp);
        if(!flag){//此时表示不是最后一个字节
            return string.substring(string.length()-8);
        }else{//此时为最后一个字节
            return string;
        }
    }

    /**
     * 对文件压缩功能的测试
     */
    @Test
    public void test2(){
        String srcFile = "D:\\Coding\\Bigdata\\exer\\JDBC基础面试题总结.docx";
        String destFile = "D:\\Coding\\Bigdata\\exer\\JDBC基础面试题总结.zip";
        zipFile(srcFile, destFile);
        System.out.println("压缩成功。");
    }
    /**
     * @Author: Mithrandir
     * @Time: 2021/2/28-20:13
     * @Description: 压缩文件：读取指定的源文件，对其进行压缩，并生成为指定的压缩文件
     * @param srcFile 要压缩的源文件
     * @param dstFile 生成的目标文件
     * @return void
     */
    public static void zipFile(String srcFile,String dstFile) {
        ObjectOutputStream oos = null;
        FileOutputStream fos = null;
        FileInputStream fis = null;
        try {
            //创建文件读入流
            fis = new FileInputStream(srcFile);
            //创建一个和源文件大小一样的byte[]
            byte[] bytes = new byte[fis.available()];
            //将源文件读取到bytes中
            fis.read(bytes);
            //采用写好的函数对bytes进行压缩
            byte[] huffmanZipBytes = huffmanZip(bytes);
            //创建文件输出流，将内存中处理好的文件进行输出
            fos = new FileOutputStream(dstFile);
            //创建一个和输出流关联的ObjectOutPutStream
            oos = new ObjectOutputStream(fos);
            //将byte[]写入到dstFile中
            oos.writeObject(huffmanZipBytes);
            //以对象流的方式将赫夫曼源码表写入是为了之后恢复文件时使用
            oos.writeObject(huffmanMap);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (oos != null) {
                try {
                    oos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (fos != null) {
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    //对所写的压缩关键函数进行阶段测试
    @Test
    public void test(){
        //要压缩的内容
        String content = "i like like like java do you like a java";
        //将要压缩的内容转为byte[]
        byte[] contentBytes = content.getBytes();
        System.out.println("原始数组："+content);
        System.out.println("原始数组的byte[]："+Arrays.toString(contentBytes));
        //对byte[]的数据进行统计
        List<Node> nodeList = getNodeList(contentBytes);
        System.out.println("对应的List<Node>："+nodeList);
        //根据统计的内容做成一个二叉树
        Node root = creatHuffmanTree(nodeList);
        System.out.println("二叉树的前序遍历：");
        root.preOrder();
        //根据二叉树，得到每个字符对应的二进制编码
        getHuffmanMap(root);
        System.out.println("Huffman编码表：" + huffmanMap);
        byte[] huffmanBytes = zip(contentBytes, huffmanMap);
        System.out.println("处理过后的byte[]："+Arrays.toString(huffmanBytes));
    }

    /**
     * @Author: Mithrandir
     * @Time: 2021/2/27-20:38
     * @param bytes 原数据的字符
     * @return byte[]  压缩之后的字符
     */
    public static byte[] huffmanZip(byte[] bytes){
        //创建一个节点队列
        List<Node> nodeList = getNodeList(bytes);
        //根据该节点队列生成一个哈夫曼树
        Node root = creatHuffmanTree(nodeList);
        //根据哈夫曼树生成对应的编码表
        getHuffmanMap(root);
        //对该哈夫曼编码进行压缩
        byte[] huffmanZipBytes = zip(bytes, huffmanMap);
        return huffmanZipBytes;
    }

    //用来存放每个字符的Huffmancode
    public static StringBuilder stringBuilder = new StringBuilder();
    //用于存放字符及对应的编码
    public static Map<Byte, String> huffmanMap = new HashMap<>();

    /**
     * @Author: Mithrandir
     * @Time: 2021/2/27-15:36
     * @param bytes 原始数组对应的bytes
     * @param huffmanMap 待压缩的哈夫曼编码
     * @return byte[] 处理过后的二进制数组
     */
    public static byte[] zip(byte[] bytes,Map<Byte,String> huffmanMap){
        StringBuilder builder = new StringBuilder();
        for (byte b:bytes){
            builder.append(huffmanMap.get(b));
        }
//        System.out.println("处理之前的byte[]："+builder);
        //计算所需的byte[]长度
        int len;
//        if(builder.length()%8==0){
//            len = builder.length()/8;
//        }else{
//            len = builder.length()/8 +1;
//        }
        len = (builder.length()+7)/8;
        //创建存储压缩后的byte数组
        byte[] huffmanCodeByte = new byte[len];
        int count = 0;
        for (int i = 0; i < builder.length(); i+=8) {
            String strbyte;
            if(i+8>=builder.length()){//如果剩余的不够8位，则直接放入huffmanCodeByte
                strbyte = builder.substring(i);
            }else{//如果足够8位，则将i及其后8位放入huffmanCodeByte
                strbyte = builder.substring(i, i+8);
            }
            huffmanCodeByte[count++] = (byte) Integer.parseInt(strbyte,2);
        }
        return huffmanCodeByte;
    }
    //对getHuffmanMap进行重载
    public static void getHuffmanMap(Node node){
        if(node!=null){
            getHuffmanMap(node.left, "0", stringBuilder);
            getHuffmanMap(node.right, "1", stringBuilder);
        }
    }
    /**
     * @param node     传入的节点
     * @param str      路径，向左是0，向右是1
     * @param sBuilder 用于拼接路径
     * @return void
     * @Author: Mithrandir
     * @Time: 2021/2/27-14:09
     */
    public static void getHuffmanMap(Node node, String str, StringBuilder sBuilder) {
        StringBuilder stringBuilder2 = new StringBuilder(sBuilder);
        //将str添加进stringBuilder中
        stringBuilder2.append(str);
        //对node进行处理
        if (node != null) {
            if (node.data == null) {//处理非叶子节点
                //递归处理左子节点
                getHuffmanMap(node.left, "0", stringBuilder2);
                //递归处理右子节点
                getHuffmanMap(node.right, "1", stringBuilder2);
            } else {//处理叶子节点
                huffmanMap.put(node.data, stringBuilder2.toString());
            }
        }
    }

    public static Node creatHuffmanTree(List<Node> list) {
        while (list.size() > 1) {
            //对list进行一个排序
            Collections.sort(list);
            //取出最小和第二小的两个node
            Node left = list.get(0);
            Node right = list.get(1);
            //根据两个较小的节点创建一个新的节点
            Node parent = new Node(null, left.weight + right.weight);
            //设置新节点和两个小节点之间的关系
            parent.left = left;
            parent.right = right;
            //将两个小节点从list中删除
            list.remove(left);
            list.remove(right);
            //将新节点加入到list中
            list.add(parent);
        }
        return list.get(0);
    }

    /**
     * @param nodes 对byte数组进行统计
     * @return java.util.List<huffmancode.Node> 返回一个由字符和对应出现次数的list
     * @Author: Mithrandir
     * @Time: 2021/2/26-20:32
     */
    public static List<Node> getNodeList(byte[] nodes) {
        //创建一个map，用于记录每个字符出现的次数
        Map<Byte, Integer> map = new HashMap<>();
        //记录次数
        for (byte b : nodes) {
            Integer count = map.get(b);
            if (map.get(b) == null) {
                map.put(b, 1);
            } else
                map.put(b, count + 1);
        }
        //以字符、出现的次数创建为一个node，并添加进入一个list
        ArrayList<Node> list = new ArrayList<>();
        for (Map.Entry<Byte, Integer> tmp : map.entrySet()) {
            list.add(new Node(tmp.getKey(), tmp.getValue()));
        }
        return list;
    }
}

class Node implements Comparable<Node> {
    Byte data;
    int weight;

    Node left;
    Node right;

    //写一个前序遍历
    public void preOrder() {
        System.out.println(this);
        if (this.left != null) {
            this.left.preOrder();
        }
        if (this.right != null) {
            this.right.preOrder();
        }
    }

    @Override
    public String toString() {
        return "Node{" +
                "data=" + data +
                ", weight=" + weight +
                '}';
    }

    public Node(Byte data, int weight) {
        this.data = data;
        this.weight = weight;
    }

    @Override
    public int compareTo(Node o) {
        return this.weight - o.weight;
    }
}
```

