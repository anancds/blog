# java的深拷贝和浅拷贝

在看ArrayList的源码的349行clone函数时，对clone函数的解释是浅拷贝。但是我觉得这个浅拷贝是分情况的，总结如下，不一定正确。
对于基本数据类型来说，下列三个成立，但是本身对象有引用对象的情况下，下列三种情况全部变成浅拷贝。

* 直接赋值（字符串外都属于浅拷贝）
* 使用构造函数（深拷贝）
* 使用clone()方法（深拷贝）

其实深拷贝还有浅拷贝的主要区别还是在于你拷贝的这个对象还有没有别的引用的对象，如果有那么对本身对象其实还是深拷贝，也就是在堆中重新创建一个对象，但是对引用对象是浅拷贝，也就是说引用对象并没有在堆中重新创建，而只是在栈中创建了引用的对象的引用而已。
所以深拷贝还有浅拷贝这个概念是针对的是别的引用对象。

再换个角度思考，对象其实是分层的，如果只有一层，那么就是深拷贝，如果是多层，就会变成浅拷贝，如果多层还想实现深拷贝，就必须给每层都实现Cloneable接口，其实更便利的方式是利用序列化来实现深拷贝。

没有引用情况，只有java基本数据结构，上述三者情况成立

    public class StringTest {
        public static void main(String[] args) {
            //字符串(不理解无colne()方法)
            String s = "sss";
            String t = s;   //深拷贝
            String y = new String(s); //深拷贝
            System.out.println("s:" + s + " t:" + t + " y:" + y);
            t = "ttt";
            y = "yyy";
            System.out.println("s:" + s + " t:" + t + " y:" + y);

            //数组  
            String[] ss = {"111", "222", "333"};
            String[] tt = ss; //浅拷贝
            String[] ww = (String[]) ss.clone();//深拷贝
            System.out.println("ss:" + ss[1] + " tt:" + tt[1] + " ww:" + ww[1]);
            tt[1] = "2t2";
            ww[1] = "2w2";
            System.out.println("ss:" + ss[1] + " tt:" + tt[1] + " ww:" + ww[1]);


            //list列表         
            ArrayList a = new ArrayList();
            for (int i = 0; i < 10; i++) {
                a.add(String.valueOf(i + 1));
            }
            ArrayList b = a;//浅拷贝
            ArrayList c = new ArrayList(a);//深拷贝
            ArrayList d = (ArrayList) a.clone();//深拷贝
            System.out
                .println("a:" + a.get(1) + " b:" + b.get(1) + " c:" + c.get(1) + " d:" + d.get(1));
            b.set(1, "bbb");
            c.set(1, "ccc");
            System.out
                .println("a:" + a.get(1) + " b:" + b.get(1) + " c:" + c.get(1) + " d:" + d.get(1));

            //HashMap
            HashMap h = new HashMap();
            h.put("1", "hhh");
            HashMap m = h;//浅拷贝
            HashMap p = new HashMap(h);//深拷贝
            HashMap n = (HashMap) h.clone();//深拷贝
            System.out.println(
                "h:" + h.get("1") + " m:" + m.get("1") + " p:" + p.get("1") + " n:" + n.get("1"));
            m.put("1", "mmm");
            p.put("1", "ppp");
            n.put("1", "nnn");
            System.out.println(
                "h:" + h.get("1") + " m:" + m.get("1") + " p:" + p.get("1") + " n:" + n.get("1"));
        }
    }

有引用的情况下，这个引用就是Employee，上述三者不成立，全部变成浅拷贝

    @SuppressWarnings("unchecked") public class ArrayListTest {
        public static void main(String args[]) {

            // deep cloning Collection in Java
            ArrayList<Employee> org = new ArrayList<Employee>();
            org.add(new Employee("Joe", "Manager"));
            org.add(new Employee("Tim", "Developer"));
            org.add(new Employee("Frank", "Developer"));

            // creating copy of Collection using copy constructor
            //        ArrayList<Employee> copy = new ArrayList(org);
            ArrayList<Employee> copy = (ArrayList<Employee>) org.clone();
            System.out.println(org);
            System.out.println(copy);

            for (Employee employee : org) {
                employee.setDesignation("staff");
            }

            System.out.println(org);
            System.out.println(copy);

        }

    }

    class Employee {
        private String name;
        private String designation;

        public Employee(String name, String designation) {
            this.name = name;
            this.designation = designation;
        }

        public String getDesignation() {
            return designation;
        }

        public void setDesignation(String designation) {
            this.designation = designation;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        @Override public String toString() {
            return String.format("%s: %s", name, designation);
        }
    }

现在用student和teacher这两个对象来理解可能更好，代码如下：

ShallowStudent

    public class ShallowStudent implements Cloneable {
        private String name;

        private int age;

        ShallowStudent(String name, int age) {
            this.name = name;
            this.age = age;
        }

        public Object clone() {
            ShallowStudent o = null;
            try {
                // Object中的clone()识别出你要复制的是哪一个对象。
                o = (ShallowStudent) super.clone();
            } catch (CloneNotSupportedException e) {
                System.out.println(e.toString());
            }
            return o;
        }

        public static void main(String[] args) {
            ShallowStudent s1 = new ShallowStudent("zhangsan", 18);
            ShallowStudent s2 = (ShallowStudent) s1.clone();
            //        s2.name = "lisi";
            //        s2.age = 20;
            //修改学生2后，不影响学生1的值。
            System.out.println("name=" + s1.name + "," + "age=" + s1.age);
            System.out.println("name=" + s2.name + "," + "age=" + s2.age);
        }
    }



ShallowStudent2

    public class ShallowStudent2 implements Cloneable {
        String name;// 常量对象。
        int age;
        Professor p;// 学生1和学生2的引用值都是一样的。

        ShallowStudent2(String name, int age, Professor p) {
            this.name = name;
            this.age = age;
            this.p = p;
        }

        public Object clone() {
            ShallowStudent2 o = null;
            try {
                o = (ShallowStudent2) super.clone();
            } catch (CloneNotSupportedException e) {
                System.out.println(e.toString());
            }
            return o;
        }

        public static void main(String[] args) {
            Professor p = new Professor("wangwu", 50);
            ShallowStudent2 s1 = new ShallowStudent2("zhangsan", 18, p);
            ShallowStudent2 s2 = (ShallowStudent2) s1.clone();
            s2.p.name = "lisi";
            s2.p.age = 30;
            System.out.println("name=" + s1.p.name + "," + "age=" + s1.p.age);
            System.out.println("name=" + s2.p.name + "," + "age=" + s2.p.age);
            //输出结果学生1和2的教授成为lisi,age为30。
            System.out.println("name=" + s1.name + "," + "age=" + s1.age);
            System.out.println("name=" + s2.name + "," + "age=" + s2.age);
        }
    }

    class Professor {
        String name;
        int age;

        Professor(String name, int age) {
            this.name = name;
            this.age = age;
        }
    }

DeepStudent

    public class DeepStudent implements Cloneable {
        String name;// 常量对象。
        int age;
        DeepProfessor p;// 学生1和学生2的引用值都是一样的。

        DeepStudent(String name, int age, DeepProfessor p) {
            this.name = name;
            this.age = age;
            this.p = p;
        }

        public Object clone() {
            DeepStudent o = null;
            try {
                o = (DeepStudent) super.clone();
                //对引用的对象也进行复制
                o.p = (DeepProfessor) p.clone();
            } catch (CloneNotSupportedException e) {
                System.out.println(e.toString());
            }
            return o;
        }

        public static void main(String[] args) {
            DeepProfessor p = new DeepProfessor("wangwu", 50);
            DeepStudent s1 = new DeepStudent("zhangsan", 18, p);
            DeepStudent s2 = (DeepStudent) s1.clone();
            s2.p.name = "lisi";
            s2.p.age = 30;
            System.out.println("name=" + s1.p.name + "," + "age=" + s1.p.age);
            System.out.println("name=" + s2.p.name + "," + "age=" + s2.p.age);
            //输出结果学生1和2的教授成为lisi,age为30。
        }
    }

    class DeepProfessor implements Cloneable {
        String name;
        int age;

        DeepProfessor(String name, int age) {
            this.name = name;
            this.age = age;
        }

        public Object clone() {
            DeepProfessor o = null;
            try {
                o = (DeepProfessor)super.clone();
            } catch(CloneNotSupportedException e) {
                System.out.println(e.toString());
            }
            return o;
        }
    }

DeepStudent2

    /**
     * 通过串行化实现深复制
     * /
    class Teacher implements Serializable {
        String name;
        int age;

        public Teacher(String name, int age) {
            this.name = name;
            this.age = age;
        }
    }


    public class DeepStudent2 implements Serializable {
        String name;//常量对象
        int age;
        Teacher t;//学生1和学生2的引用值都是一样的。

        public DeepStudent2(String name, int age, Teacher t) {
            this.name = name;
            this.age = age;
            this.t = t;
        }

        public Object deepClone() throws IOException, ClassNotFoundException {//将对象写到流里
            ByteArrayOutputStream bo = new ByteArrayOutputStream();
            ObjectOutputStream oo = new ObjectOutputStream(bo);
            oo.writeObject(this);//从流里读出来
            ByteArrayInputStream bi = new ByteArrayInputStream(bo.toByteArray());
            ObjectInputStream oi = new ObjectInputStream(bi);
            return (oi.readObject());
        }

        public static void main(String[] args) throws IOException, ClassNotFoundException {
            Teacher t = new Teacher("tangliang", 30);
            DeepStudent2 s1 = new DeepStudent2("zhangsan", 18, t);
            DeepStudent2 s2 = (DeepStudent2) s1.deepClone();
            s2.t.name = "tony";
            s2.t.age = 40;
            //学生1的老师不改变
            System.out.println("name=" + s1.t.name + "," + "age=" + s1.t.age);
        }
    }
