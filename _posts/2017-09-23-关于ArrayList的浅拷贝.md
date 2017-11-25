关于ArrayList的浅拷贝
---

    public static void main(String[] args){
        ArrayList<Integer> list=new ArrayList<Integer>();
        list.add(1);
       ArrayList<Integer> arrayList=(ArrayList<Integer>)list.clone();
        list.set(0,1);
        System.out.println(list.get(0));
        System.out.println(arrayList.get(0));

    }
 对于这段代码，发现两者输出值不同，似乎与ArrayList的浅拷贝不符。当存入对象时,发现更改其中一个值会影响另一个值。
 ---
    private String name;

    public Test(String name){
        this.name=name;
    }

    public void setName(String name){
        this.name=name;
    }

    public String getName(){
        return name;
    }
    public static void main(String[] args){
        ArrayList<Test> list=new ArrayList<Test>();
        Test test=new Test("cs1");
        list.add(test);
        ArrayList<Test> copyList=(ArrayList<Test>)list.clone();
        test.setName("cs2");
        System.out.println(list.get(0).getName());
        System.out.println(copyList.get(0).getName());
    }
可以推断，对于整型数据，当整数范围在-128～127之间时，整数存放于运行时常量池中，当修改list的整数时，则是将母本的引用指向新的引用；当整数范围超过这个范围时，则通过Integer.valeOf自动装箱新创建一个整数对象，将母本的引用指向新的整数，故而母本的改变不影响副本。下面的这一情况为母本和副本引用同一个对象，当任意其一改变这个对象时，另一个都会受到影响。

现总结下ArrayList深拷贝的集中方法。
1. 进行for循环，数组中的每个对象都进行clone，另对象对应的class需实现Clonable接口

---
    private String name;

    public Test(String name){
        this.name=name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public static void main(String[] args){
        ArrayList<Test> list=new ArrayList<Test>();
        list.add(new Test("cs1"));
        ArrayList<Test> copyList=new ArrayList<Test>();
        for(Test test:list){
            try {
                copyList.add((Test) test.clone());
            } catch (CloneNotSupportedException e) {
                e.printStackTrace();
            }
        }
        Test copyTest=list.get(0);
        copyTest.setName("cs2");
        System.out.println(list.get(0).getName());
        System.out.println(copyList.get(0).getName());

    }
2. 采用序列化进行clone，则对象所在的class需实现Serializable

---
    private String name;

    public Test(String name){
        this.name=name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        ArrayList<Test> list=new ArrayList<Test>();
        Test test=new Test("cs1");
        list.add(test);
        ByteArrayOutputStream byteOut=new ByteArrayOutputStream();
        ObjectOutputStream out=new ObjectOutputStream(byteOut);
        out.writeObject(list);
        out.flush();
        out.close();
        ByteArrayInputStream byteIn=new ByteArrayInputStream(byteOut.toByteArray());
        ObjectInputStream in=new ObjectInputStream(byteIn);
        ArrayList<Test> copyList=(ArrayList<Test>)in.readObject();
        test.setName("cs2");
        System.out.println(list.get(0).getName());
        System.out.println(copyList.get(0).getName());

    }



