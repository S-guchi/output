# 単純ファクトリーとFactory Methodについて

## 単純ファクトリーパターン

条件分岐のついた生成メソッドを持つクラスのこと。
渡されたオブジェクトの種類や引数によって作るインスタンスを変える。
メリットはわかりやすいことと、似たようなオブジェクトの生成ロジックをまとめることができること。
デメリットは、オブジェクトの種類や引数が増えていくと、共にクラスが巨大化していくこと。
引数で切り替えるオブジェクトを管理する必要があること。
（手を入れる際はメソッドの中身と引数を照らし合わせてどんなオブジェクトが作られるか把握する必要がある）

これはGoFのデザインパターンには含まれていない。

### 単純ファクトリーパターンのクラス図

```mermaid
classDiagram
    class Human {
        +speak()
    }

    class Child {
        +speak()
    }

    class Adult {
        +speak()
    }

    class HumanFactory {
        +create_human(human_type)
    }

    HumanFactory --> Human
    Human <|-- Child
    Human <|-- Adult
```

### 単純ファクトリーパターンの具体的なコード

人間オブジェクトを作るメソッドを持ったクラス
大人 or 子供 くらいであれば単純ファクトリーで良さそう

```mermaid
classDiagram
  HumanFactory <|.. Child
  HumanFactory <|.. Adult


  HumanFactory : +create_human(human_type)
  Child : +speak()
  Adult : +speak()
```

```python
from abc import ABC, abstractmethod

class Human(ABC):
    @abstractmethod
    def speak(self):
        pass

class Child(Human):
    def speak(self):
        print("子供です！！！！")

class Adult(Human):
    def speak(self):
        print("大人です！！！！")


class HumanFactory:
    def create_human(human_type):
        if human_type == "子供":
            return Child()
        if human_type == "大人":
            return Adult()

human = HumanFactory.create_human("子供")
human.speak() # 子供です！！！！
```

新しく人間オブジェクトを増やそうかな、create_humanメソッドに手を入れることになるが
オブジェクトの種類が増えると、以下のように複雑になりがち

```mermaid
classDiagram
  HumanFactory <|.. Baby
  HumanFactory <|.. Child
  HumanFactory <|.. Junior
  HumanFactory <|.. Highschool
  HumanFactory <|.. Adult
  HumanFactory <|.. Adult2
  HumanFactory <|.. ojiisan
  HumanFactory <|.. obaasan

  HumanFactory : create_human(human_type, sex, age)
  ```

```python
class HumanFactory:
    def create_human(human_type, sex, age):
        if human_type == "赤ちゃん":
            return Baby()
        if human_type == "子供":
            return Child()
        if human_type == "中学生":
            return Junior()
        if human_type == "高校生":
            return Highschool()
        if human_type == "大人":
            if sex == "男":
                return Adult()
            if sex == "女":
                return Adult2()
        if age >= 60 and sex == "男":
            return ojiisan()
        if age >= 60 and sex == "女":
            return obaasan()

human = HumanFactory.create_human("年配", "男", 60)
human.speak() # おじいさんです！！！！
```

時間が経つにつれて、オブジェクトの種類が増えていくと、管理が大変になってくる。
その時になって初めて、GoFのファクトリーメソッドパターンやAbstractFactoryパターンを使うと良さそう。
もちろん、作るオブジェクトが最初から増える見込みであればファクトリーメソッドやAbstractFactoryパターンを使ってもいいと思う。

(調べたところ現実ではこの単純ファクトリーでの実装が多いとのこと、やっぱりわかりやすいから？)

## ファクトリーメソッドパターン

Gofの生成デザインパターンの一つ。
抽象クラスで生成するオブジェクトのインターフェースを設定し、それを元にサブクラスで具体的なオブジェクトを生成する。

### ファクトリーメソッドパターンのクラス図

```mermaid
classDiagram

    class HumanFactory {
        +create_human(): Human
    }

    class ChildFactory {
        +create_human(): Child
    }

    class AdultFactory {
        +create_human(): Adult
    }

    HumanFactory <|-- ChildFactory
    HumanFactory <|-- AdultFactory
```
```mermaid
classDiagram


    class HumanFactory {
        +create_human(): Human
    }

    class BabyFactory {
        +create_human(): Baby
    }

    class ChildFactory {
        +create_human(): Child
    }

    class JuniorFactory {
        +create_human(): Junior
    }

    class HighschoolFactory {
        +create_human(): Highschool
    }

    class AdultFactory {
        +create_human(): Adult
    }

    class OjiisanFactory {
        +create_human(): Ojiisan
    }

    class ObaasanFactory {
        +create_human(): Obaasan
    }

    HumanFactory <|-- BabyFactory
    HumanFactory <|-- ChildFactory
    HumanFactory <|-- JuniorFactory
    HumanFactory <|-- HighschoolFactory
    HumanFactory <|-- AdultFactory
    HumanFactory <|-- OjiisanFactory
    HumanFactory <|-- ObaasanFactory
```

## ファクトリーメソッドパターンの具体的なコード

```python
from abc import ABC, abstractmethod

class Human(ABC):
    @abstractmethod
    def speak(self):
        pass

class HumanFactory(ABC):
    @abstractmethod
    def create_human(self):
        pass

class BabyFactory(HumanFactory):
    def create_human(self):
        return Baby()

class ChildFactory(HumanFactory):
    def create_human(self):
        return Child()

class JuniorFactory(HumanFactory):
    def create_human(self):
        return Junior()

class HighschoolFactory(HumanFactory):
    def create_human(self):
        return Highschool()

class AdultFactory(HumanFactory):
    def __init__(self, sex):
        self.sex = sex

    def create_human(self):
        if self.sex == "男":
            return Adult()
        else:
            return Adult2()

class OjiisanFactory(HumanFactory):
    def __init__(self, sex, age=60):
        self.sex = sex
        self.age = age

    def create_human(self):
        if self.age >= 60 and self.sex == "男":
            return Ojiisan()
        return None

class ObaasanFactory(HumanFactory):
    def __init__(self, sex, age=60):
        self.sex = sex
        self.age = age

    def create_human(self):
        if self.age >= 60 and self.sex == "女":
            return Obaasan()
        return None

def get_human_factory(human_type, sex=None, age=None):
    if human_type == "赤ちゃん":
        return BabyFactory()
    if human_type == "子供":
        return ChildFactory()
    if human_type == "中学生":
        return JuniorFactory()
    if human_type == "高校生":
        return HighschoolFactory()
    if human_type == "大人":
        return AdultFactory(sex)
    if human_type == "おじいさん":
        return OjiisanFactory(sex, age)
    if human_type == "おばあさん":
        return ObaasanFactory(sex, age)

```

## まとめ

調べたところ、単純ファクトリーは「悪」ということではなく、むしろこちらがファクトリーメソッドパターンより使われていそう。

1. ファクトリーパターンで作ったクラスが大きくなってしまった！！
1. 大きすぎて読みづらいから、クラス内のメソッドをサブクラスに移して読みやすいようにしよう。
1. あれ？2を繰り返している間にいつの間にかFactoryMethodになっているね！いい感じだね！

という流れでのリファクタリングの最終ゴールがFactoryMethodみたい。
リファクタリングの指標の一つと覚えておけばよさそう。

## 参考

デザインパターン「Factory Method」
<https://qiita.com/shoheiyokoyama/items/d752834a6a2e208b90ca>

デザインパターン入門Factory Methodパターンについて
<https://zenn.dev/komorimisaki422/articles/21e40f13f514ae#5.-factory-method%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3%E3%81%AE%E3%81%BE%E3%81%A8%E3%82%81>>

ファクトリーの比較
<https://refactoring.guru/ja/design-patterns/factory-comparison>
