# Template

- 实例化

  - 显示实例化

    对于模板函数在使用当中类型确定的过程，例如：

    ```c++
    template <typename T>
    void Swap(T &a, T &b)
    ```

    第一种方式是声明所需的种类，用 <> 符号来指示类型，并在声明前加上关键词 template，如下：

    ```c++
    template void Swap<int>(int &, int &);
    ```

    第二种方式是直接在程序中使用函数创建，如下：

    ```c++
    Swap<int>(a,b);
    ```

    显式实例化直接使用了具体的函数定义，而不是让程序去自动判断。

  - 隐式实例化

    隐式实例化比较简单，就是最正常的调用，Swap(a,b)，直接导致程序生成一个 Swap() 的实例，该实例使用的类型即参数 a 和 b 的类型，编译器根据参数来定义函数实例。

- 具象化

  思考这么一个问题，当前的Swap模板交换输入的两个对象，可能式基本类型也可能式自定义类。如果有这么一个需求，需要交换自定义类里的某一个属性而不是整个类，那么Swap模板就不可用，因为Swap模板交换的是整个类。

  显式具体化将不会使用Swap()模板来生成函数定义，而应使用专门为该特定类型显式定义的函数类型。有两种定义形式，如下，其中job为用户自定义类

  ```c++
  template <> void Swap(job &a, job &b)
  template <> void Swap<job>(job &a, job &b)
  ```

  显式具体化在声明后，必须要有具体的实现，这是与显示实例化不同的地方

- 实例（实例化与具体化）

  ```c++
  /*************************************************************************
  	> File Name: array.cpp
  	> Author: 
  	> Mail: 
  	> Created Time: 
   ************************************************************************/
  #include<iostream>
  #include<string>
  using namespace std;
   
  struct job
  {
      string name;
      int salary;
      job(string _name,int _salary):name(_name),salary(_salary){};
  };
   
  //template prototype
  template <typename T>
  void Swap(T &a, T &b){
      T temp;
      temp = a;
      a = b;
      b = temp;
  }
   
  //explict specialization for job 显式具体化
  template <> void Swap(job &a, job &b)
  {
      int temp;
      temp = a.salary;
      a.salary = b.salary;
      b.salary = temp;
  }
   
  template void Swap<int>(int &, int &);//显示实例化
  int main()
  {
      char a = 'a', b = 'b';
      cout<<"a: "<<a<<" ; b: "<<b<<endl;
      Swap(a,b);  //1 implicit template instantiation for char 隐式实例化
      cout<<"a: "<<a<<" ; b: "<<b<<endl;
   
      int c = 1, d = 2;
      cout<<"c: "<<c<<" ; d: "<<d<<endl;
      Swap(c,d);  //2 use explicit template instantiation for int 显式实例化
      cout<<"c: "<<c<<" ; d: "<<d<<endl;
     
      Swap<int>(c,d);  //3 use explict template instantiation for int 显式实例化
      cout<<"c: "<<c<<" ; d: "<<d<<endl;
      
      job e("lucy",100), f("bob",200);
      cout<<"lucy: "<<e.name<<" "<<e.salary<<" ; bob: "<<f.name<<" "<<f.salary<<endl;
      Swap(e,f);  //use explict specialization for job 调用显式具体化
      cout<<"lucy: "<<e.name<<" "<<e.salary<<" ; bob: "<<f.name<<" "<<f.salary<<endl;
  }
  ```

  输出

  ```c++
  ➜  cpptest ./a.out      
  a: a ; b: b
  a: b ; b: a
  c: 1 ; d: 2
  c: 2 ; d: 1
  c: 1 ; d: 2
  lucy: lucy 100 ; bob: bob 200
  lucy: lucy 200 ; bob: bob 100
  ```

  

  

  

  

  

  