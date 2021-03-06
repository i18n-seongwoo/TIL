# 스칼라로 배우는 함수형 프로그래밍

스칼라로 배우는 함수형 프로그래밍 책을 보고 내용을 정리하는 페이지다.


## 2장 연습문제 풀이 ##

### 2.2 ###
#### 문제 ####
Array[A]가 주어진 비교함수에 의거하여 정렬되어 있는지를 체크하는 isSorted[A]를 구현해라
```
  def isSorted[A](as: Array[A], ordered: (A, A) => Boolean): Boolean
```

#### 풀이 ####
* tailrec annotation을 사용해서 tail recursion을 사용하는지 체크했다
* loop 내장함수를 사용해서 루프를 구현했다.
```
import scala.annotation.tailrec

object ArrayUtils {
  def isSorted[A](as: Array[A], ordered: (A, A) => Boolean): Boolean = {
    @tailrec
    def loop(n: Int, acc: Boolean): Boolean = 
      if (n == as.length - 1) acc 
      else loop(n + 1, acc && ordered(as(n), as(n + 1)))
    loop(0, true)
  }

  def main(args: Array[String]) = {
    val arrs:Array[Int] = Array[Int](1,2,3,4,5,6,7)
    val notSorted:Array[Int] = Array[Int](1,2,4,3,5,6,7)
    val sorted:Array[Int] = Array[Int](1,2,4,4,5,5,7)
    
    println(isSorted[Int](arrs, {(a, b) =>
            a <= b 
          }))
    println(isSorted[Int](notSorted, ((a, b) => a <= b )))
    println(isSorted[Int](sorted, ((a, b) => a <= b )))
  }
}
```


### 2.3, 2.4,2.5 ###
#### 문제 ####
curry , uncurry, compose를 구현해라
```
  def curry[A, B, C](f: (A, B) => C): A => (B => C) 
  def uncurry[A, B, C](f: A => B => C): (A, B) => C 
  def compose[A, B, C](f: B => C, g: A => B): A => C
```
#### 풀이 ####
* 고차 함수의 특성을 이용했다.

```
object Curry extends App {
  def curry[A, B, C](f: (A, B) => C): A => (B => C) =
    (a: A) => (b: B) => f(a, b)

  def uncurry[A, B, C](f: A => B => C): (A, B) => C =
    (a: A, b: B) => f(a)(b)

  def compose[A, B, C](f: B => C, g: A => B): A => C =
    (a: A) => f(g(a))

  println(curry[Int, Int, Int]((a: Int, b: Int) => a + b)(10)(20))
  println(uncurry[Int, Int, Int] { a: Int => (b: Int) => a + b }(10, 20))
  println(compose[Int, Int, Int]({ a: Int => a * 10 }, { b: Int => b + 1 })(2))
}
```

## 3장 함수적 자료구조 정리 ##
* 함수적 자료구조는 immutable 이다.
* 그리고 자료구조 내에서 같은 값의 자료는 공유한다. (일종의 design pattern 중 flyweight 패턴과 비슷)

### sealed 키워드 ###
* 클래스나 trait 앞에 붙는다
```
sealed trait Gender
object Male extends Gender
object Female extends Gender
```
* 위와 같이 사용한다. 
* 상속을 같은 파일 내에서만 하도록 제한할 때 사용한다.

### 공변(covariant) ###
* generic에 [+A], [-A], [A] 3가지가 있다.
* [+A]는 컴파일러가 지정한 타입 하위 타입으로 간주한다
```
 val list:List[Animal] = List[Dog]()
```
* [A]는 어떤 타입이든지 개의치 않으며..
* [-A]는 지정한 타입 상위 타입으로 간주 한다는 것이다.


### 패턴 매칭 ###
* companion 객체는 패턴 매칭을 사용한다.
* 패턴 매칭 규칙은 식별자와 자료 생성자로 구성된다. 이 모든 것이 매칭되는 케이스가 있으면, 그걸 수행한다.
* 구조적으로 동등하면 매칭된다고 본다.
* 생성자 패턴과 변수를 함께 묶는다. 
* 매치가 안되면? MatchError 오류를 발생한다. 
* 예
```
 def sum(ints: List[Int]): Int = ints match {
   case Nil => 0
   case Cons(x, xs) => x + sum(xs)
 }
 
 def product(ds: List[Double]): Double = ds match {
   case Nil => 1.0
   case Cons(0.0, _) => 0.0
   case Cons(x, xs) => x * product(xs)
 }
 
 List (1,2,3) match { case Cons(h, _) => h }
 // 위와 상등은            Cons(1, Cons(2, Conms(3, Nil))) 이다
 // 즉 위 값은 1이다.
```
### 대수적 자료 형식 ###
* 일종의 복합 타입이다.(Tuple이나 record가 예)
* 1개 이상의 data 생성자만 있다. 데이터 생성자의 파라미터는 필드라 부른다.
* 이런 data 형식을 data 생성자의 sum합 또는 합집합union이라 부른다.
* 각 data 생성자는 생성자가 가진 인수의 곱product 이라 부른다.
* type 선언이 귀찮으면 커링을 써서 분리하면 컴파일러가 쉽게 타입을 판단한다

### 참고 ###

* [대수적자료형식 위키피디아](https://en.wikipedia.org/wiki/Algebraic_data_type)


### 3.3 문제 풀이 ###
#### 문제 ####
* drop, dropWhile을 구현하라.
* 첫 요소부터 시작한다
* f가 true 일때까지 요소를 지우고 돌려준다( dropWhile만)
#### 풀이 ####
```
package fpinscala.datastructures

sealed trait List[+A]
case object Nil extends List[Nothing]
case class Cons[+A](head: A, tail: List[A]) extends List[A]

object List {
  def apply[A](as: A*): List[A] =
    if (as.isEmpty) Nil
    else Cons(as.head, apply(as.tail: _*))

  def tail[A](l: List[A]): List[A] = l match {
    case Nil        => Nil
    case Cons(h, t) => t
  }

  def drop[A](l: List[A], n: Int): List[A] = {
    def go(next: List[A], cnt: Int): List[A] = 
      if (cnt == n || next == Nil) next
      else go(tail(next), cnt + 1)    
    go(l, 0)
  }  

  def dropWhile[A](l: List[A], f: A => Boolean): List[A] = l match {
    case Nil        => Nil
    case Cons(h, t) => if (f(h)) dropWhile(tail(l), f) else l
  }
}

object ListTest extends App {
  println(List.dropWhile[Int](List(1, 1, 3, 4), _ % 2 != 0))
  println(List.dropWhile[Int](List(1, 2, 3, 4), _ % 2 != 0))
}
```

### 3.6 문제풀이 
#### 문제
* 맨 뒤 요소를 빼는 init을 구현한다. 

#### 해결책
* O(N)으로 밖에 구현이 안된다. 
* 링크드 리스트고 탐색하는데 걸리는시간이 O(N)이기 때문이다. 
```
  def init[A](l: List[A]): List[A] = l match {
    case Nil => Nil
    case Cons(_,Nil) => Nil
    case Cons(h, t) => Cons(h, init(t))
  }
```

### 3.7
#### 문제
* fold로 구현한게 0.0을 만났을때 재귀를 멈추는가?
#### 해결책
* 구현하지 않았다. 
* 0을 만났을 때 fold를 중단하는 방식을 찾으면 된다.

### 3.9
#### 문제
* length를 구현하라
#### 해결책
```
def length[A](ns: List[A]) =
    foldRight[A, Int](ns, 0)((x: A, y: Int) => y + 1)

```
### 3.10
#### 문제
* foldLeft를 구현하라
* stack safe하도록 구현해라

#### 해결책
* Tail Recursion과 가산 변수를 메소드에 추가하자
* 이러면 가산 변수에 결과값을 넣으면 tail recursion을 한다.
```
  def foldLeft2[A, B](as: List[A], z: B)(f: (B, A) => B): B = {
    @tailrec
    def fold[A, B](l: List[A], b: B, fx: (B, A) => B, acc: B): B = l match {
      case Nil          => b
      case Cons(h, Nil) => fx(acc, h)
      case Cons(h, t)   => fold(t, b, fx, fx(acc, h))
    }
    fold(as, z, f, z)
  }
```

### 3.11
#### 문제
* sum, product, length를 foldLeft로 구현해라

#### 해결책
```
  def sumByFoldLeft(ns: List[Int]): Int =
    foldLeft2(ns, 0)(_ + _)

  def productByFoldLeft(ns: List[Double]): Double =
    foldLeft2(ns, 1.0)(_ * _)
    
  def lengthByFoldLeft[A](ns: List[A]): Int = 
    foldLeft2[A, Int](ns, 0)((x: Int, y: A) => x + 1)
    
```

### 3.12 
#### 문제
* reverse 를 구현해라. 
* 단 fold메소드를 사용해서 구현할 것

#### 해결책 

* 빈 리스트 (apply로 문제 해결)
```
  def reverse[A](as: List[A]): List[A] = 
    foldLeft(as, List[A]())((acc,el) => Cons(el, acc))
```

### 3.13
#### 문제
* FoldLeft를 foldRight로 구현해라
 
#### 해결책
* reverse로 뒤집는다.
```
  def foldRight2[A, B](as: List[A], z: B)(f: (A, B) => B): B =
    foldLeft(reverse(as), z)((b, a) => f(a, b))
```
### 3.16
#### 문제
* 정수 List에서 1을 더해서 목록을 변환하는 함수를 작성해라
* 단 새 List를 돌려주어야 한다.
#### 해결책
```
  def plusOne(l: List[Int]): List[Int] =
    foldRight(l, List[Int]())((a, b) => Cons(a + 1, b))
```

### 3.17
#### 문제
* List[Double]을 String으로 변환하는 함수를 작성해라 

#### 해결책
* foldRight를 활용하여 문제를 풀었다.
```
  def convertDoubleToString(l: List[Double]): List[String] =
    foldRight(l, List[String]())((a, b) => Cons(a.toString(), b))
```

### 3.18
#### 문제
* map을 구현해라
```
def map[A, B](l: List[A])(f: A => B): List[B]
```
#### 해결책
```
import scala.collection.mutable.ListBuffer
  def map[A, B](l: List[A])(f: A => B): List[B] = l match {
    case Nil        => Nil
    case Cons(h, t) => Cons(f(h), map(t)(f))
  }
  
  def map2[A, B](l: List[A])(f: A => B): List[B] = {
    val buffer = new ListBuffer[B]()
    def go(l: List[A]): Unit = l match {
      case Nil => ()
      case Cons(h, t) => {
          buffer += f(h)
          go(t)
      }
    }
    go(l)
    List(buffer.toList: _*)    
  }
```
### 3.19
#### 문제
* filter를 구현해라
```
def filter[A](l: List[A])(f: A => Boolean): List[A]
```
#### 해결책
* ListBuffer 또는 FoldRight를 이용해서 구현한다.
```
import scala.collection.mutable.ListBuffer
  def filter[A](l: List[A])(f: A => Boolean): List[A] = {
    val buffer = new ListBuffer[A]()
    def go(as: List[A]): Unit = as match {
      case Nil => ()
      case Cons(h, t) => {
        if (!f(h)) buffer += h
        go(t)
      }
    }
    go(l)
    List(buffer.toList: _*)
  }

  def filterViaFoldRight[A](l: List[A])(f: A => Boolean): List[A] =
    foldRight(l, Nil: List[A])((h, t) => if (f(h)) Cons(h, t) else t)
```
### 3.20
#### 문제
* flatMap을 구현해라
```
 List.flatMap(List(1,2,3))(i => List(i, i))
```

* 아래는 함수 시그니쳐이다. 
```
  def flatMap[A, B](l: List[A])(f: A => List[B]): List[B]
```
#### 해결책
* 리스트와 리스트를 붙여주는 append 를 이용한다
```
  def flatMap[A, B](l: List[A])(f: A => List[B]): List[B] = l match {
    case Nil        => Nil
    case Cons(h, t) => append(f(h), flatMap(t)(f))
  }
```
### 3.21
#### 문제
* filter를 이용해서 flatMap을 구현해라
```
def filterViaFlatMap[A](l: List[A])(f: A => Boolean): List[A]
```
#### 해결책
```
  def filterViaFlatMap[A](l: List[A])(f: A => Boolean): List[A] =
    flatMap(l)(v => if (f(v)) Nil else List(v))  
```


### 3.22
#### 문제
* 두 리스트를 더하는 메소드를 만들어라
* List(1,2,3) 와 List(4,5,6) 을 더하면  List(5,7,9)라는 결과가 나오는 메소드를 만들어라 

#### 해결책
* 패턴 매치를 두 개 인자를 대상으로 사용할 수 있다.
```
  def plusLists(l: List[Int], r: List[Int]): List[Int] = (l, r) match {
    case (Nil, Nil)                   => Nil
    case (Cons(h, t), Nil)            => Cons(h, t)
    case (Nil, Cons(h, t))            => Cons(h, t)
    case (Cons(lh, lt), Cons(rh, rt)) => Cons(lh + rh, plusLists(lt, rt))
  }
```

### 3.23

#### 문제
* 3.22를 일반화 시켜서 zipWith 메소드를 만들어라 

#### 해결책
* 커링을 이용해 연산용 메소드를 받는다.
```
  def zipWith[A, B, C](l: List[A], r: List[B])(f: (A, B) => C): List[C] = (l, r) match {
    case (Nil, Nil)                   => Nil
    case (Cons(h, t), Nil)            => Nil
    case (Nil, Cons(h, t))            => Nil
    case (Cons(lh, lt), Cons(rh, rt)) => Cons(f(lh, rh), zipWith(lt, rt)(f))
  }
```
### 3.24

#### 문제
* List가 또다른 List를 부분 순차열로 담고 있는지 체크하는 메소드를 구현한다
* 예를 들어 List(1,2,3,4)는 List(4), List(1,2) 등이 부분 순차열이다.

#### 해결책
* 한 번에 모두 풀려고 하지 말자
* sup 파라미터의 현재 위치에서 주어진 부분 순차열(sub)에 포함되는가의 메소드(loop)
* go는 sup을 뒤로 조금씩 이동하면서 loop가 true를 돌려주는지를  체크한다. 
* 개선점이 있다면 loop나 go는 내부 함수로 두지 않아도 된다.
* case () if를 이용해서 특정 조건일때만 case를 수행되게 하고 나머지는 false처리 할 수 있다.
* 가산 변수 acc를 사용하지 않아도 된다.
```
  def hasSubsequence[A](sup: List[A], sub: List[A]): Boolean = {
    @tailrec
    def loop(target: List[A], subList: List[A], acc: Boolean): Boolean = (target, subList) match {
      case (_, Nil)                     => acc
      case (Cons(lh, lt), Cons(rh, rt)) => if (acc) loop(lt, rt, lh == rh && acc) else acc
      case _                            => false
    }

    @tailrec
    def go(l: List[A]): Boolean = l match {
      case Nil        => false
      case Cons(_, t) => if (loop(l, sub, true)) true else go(t)
    }
    go(sup)
  }
```
* 개선 풀이

```
  def hasSubsequence[A](sup: List[A], sub: List[A]): Boolean = {
    @tailrec // acc 제거 
    def loop(target: List[A], subList: List[A]): Boolean = (target, subList) match {
      case (_, Nil)                                 => true
      case (Cons(lh, lt), Cons(rh, rt)) if lh == rh => loop(lt, rt)
      case _                                        => false
    }

    @tailrec
    def go(l: List[A]): Boolean = l match {
      // subList와 리스트가 같은 값으면 true
      case Nil               => sub == l
      // l이 어떤 값이 오든 현재의 l이 sub로 시작한다면 같은 값 
      case _ if loop(l, sub) => true
      // 다른 값이라 넘어감
      case Cons(_, t)        => go(t)
    }
    go(sup)
  }
```

### 모범답안
```
@annotation.tailrec
def startsWith[A](l: List[A], prefix: List[A]): Boolean = (l,prefix) match {
  case (_,Nil) => true
  case (Cons(h,t),Cons(h2,t2)) if h == h2 => startsWith(t, t2)
  case _ => false
}
@annotation.tailrec
def hasSubsequence[A](sup: List[A], sub: List[A]): Boolean = sup match {
  case Nil => sub == Nil
  case _ if startsWith(sup, sub) => true
  case Cons(_,t) => hasSubsequence(t, sub)
}
```
### 3.25

#### 문제
* 트리의 전체 개수를 구해라
#### 풀이 
```
  def size[A](tree: Tree[A]): Int = tree match {
    case Leaf(v)      => 1
    case Branch(l, r) => 1 + size(l) + size(r)
  }
```

### 3.26
#### 문제
* 가장 큰 Leaf의 value를 구해라
#### 풀이
```
  def maximum(tree: Tree[Int]): Int = tree match {
    case Leaf(v)      => v
    case Branch(l, r) => maximum(l) max maximum(r)
  }
```

### 3.27
#### 문제
* tree의 depth를 구해라
#### 풀이
```
  def depth[A](tree: Tree[A]): Int = tree match {
    case Leaf(_)      => 0
    case Branch(l, r) => 1 + depth(l).max(depth(r))
  }
```
## 4 장 예외를 이용하지 않는 오류처리
### 예외의 단점
```
def failingFn(i: Int): Int = { 
  val y: Int = throw new Exception("fail!")
  try {
    val x = 42 + 6
    x + y
  } catch { case e: Exception => 43 }
}
```
* 예외는 참조 투명성을 위반한다
- 참조 투명한 표현식은 지역적으로 추론이 가능함. 즉, 이 코드만 보고도 뭔 내용인지 안다라는 뜻.
- 예외를 사용하면 코드리딩을 할때 호출하는 곳으로 이동해야 어떤 처리를 하는지 알수 있다.
* 문맥 의존성을 도입한다.
* 형식에 안전하지 않다. Int => Int 인자와 리턴타입만 보곤 예외를 던진다는걸 모른다.

### 장점
* 중앙집중화
* 오류 처리 논리의 통합

### checked Exception
* 결과를 미리 체크해서 예외 결과를 리턴 값으로 주는 경우
* 고차 함수에서는 통하지 않는다!(인자로 받는 함수안에서 뭔 Exception을 던질지 모른다.)

### 예외의 대안

#### 가짜값 돌려주기
* 0/0 같은 경우는 0을 돌려준다
* 그러나 이 책에선 이런 접근 방식을 피한다
* 오류가 아무런 알림없이 전파된다. 컴파일러가 경고를 해주지 않는다.
* 호출자쪽에서 진짜 결과를 받았는지 점검하는 비스무리한 코드가 늘어난다.
* 다형적 코드에는 적용할 수 없다
* 호출자에 특별한 방침이나 호출 규약이 생긴다.

#### 처리할 수 없는 경우의 default값을 돌려주기
* 완전한 함수가 된다
* 하지만 이 함수의 처리 방식을 호출자가 알고 있어야 한다.

#### 새로운 자료형식의 도입 Option
* 반환 형식을 명시적으로 표현하는 것.

```
sealed trait Option[+A]
case class Some[+A] extends Option[A]
case object None extends Option[Nothing]

def mean(xs:Seq[Double]): Option[Double] = if (xs.isEmpty) None else Some(xs.sum / xs.length)
```

* 위 예제는 모든 값에 하나의 리턴 타입을 가진다.
* 유효하지 않는 출력은 None을 돌려주기 때문

##### Option 사용 패턴
* Map에서 주어진 키를 찾는 함수
* List와 iterable 형식에 headOption, lastOption 은 순차열이 비지 않는 경우 첫 요소나 마지막 요소를 가진 Option을 돌려줌

##### Option의 기본 함수 
```
trait Option[+A] {
  def map[B](f: A => B): Option[B]
  def flatMap[B](f: A => Option[B]): Option[B]
  def getOrElse[B >: A](default: => B): B
  def orElse[B >: A](ob: => Option[B]): Option[B]
  def filter(f: A => Boolean): Option[A]
}
```
* B >: A B가 A타입이거나 A의 상위 타입이어야 한다는 의미
* default: => B 인수가 B 지만 평가되기전까지는 사용하지 않는다.


### 문제 풀이 

#### 4.1

##### 문제 

* Option의 기본 함수를 구현하고 어떻게 쓰일지 상상해보라
* map, getOrElse는 패턴 부합을 쓰고 나머지는 쓰지 말아라

##### 풀이
```
  def map[B](f: A => B): Option[B] = this match {
    case Some(a) => Some[B](f(a))
    case _ => None
  }
  def flatMap[B](f: A => Option[B]): Option[B] = map(f) getOrElse None
  
  def getOrElse[B >: A](default: => B): B = this match {
    case Some(a) => a
    case _ => default
  }
  def orElse[B >: A](ob: => Option[B]): Option[B] = this map (Some(_)) getOrElse ob
  
  def filter(f: A => Boolean): Option[A] = flatMap(a => if(f(a)) Some(a) else None) 
```

#### 4.2
##### 문제 
* Option의 flatMap과 mean을 이용해서 variance(분산)을 구해라
* Seq에 대해 math.pow(x - m, 2)의 평균이 분산이다.
* 아래는 시그니쳐
```
def variance(xs: Seq[Double]): Option[Double] 
```
##### 풀이

```
object Math {
  def mean(xs: Seq[Double]): Option[Double] = if (xs.isEmpty) None else Some(xs.sum / xs.length)
  // 4.2 
  def variance(xs: Seq[Double]): Option[Double] = mean(xs).flatMap { m => mean(xs.map(x => math.pow(x - m, 2))) }
}
```
### Option의 기본적인 함수 용례

#### map 
*  option 안에 결과가 있다면 변환하는데 사용. 오류가 없다는 가정하에 계산할 수 있다.
* 술어함수가 실패하면 None을 준다. 

#### flatMap 
*  여러 단계의 계산을 진행할때 유용하다. 중간 단계가 실패하면 계산 과정을 모두를 취소한다.
* 술어함수가 실패하면 None을 준다

#### filter
* filter 는 주어진 술어 함수가 false 일때 요소를 제거한다.

그래서 
```
val dept: String = lookupByName("json").map(_.dept).filter(_ != "Accounting").getOrElse("Default Dept")
```
위에 처럼 일반 imperative language 에선 상상도 할 수 없는 일을 할 수 있다.

#### getOrElse
값을 구할 수 없을 때  아래 idiom을 사용한다
```
getOrElse(throw new Exception ("Failed to calculate")) 
```
단 예외를 잡을 수 없는 상황에서만 사용한다

* 예외 처리를 뒤로 미루는 매커니즘.

### 예외의 승급과 감싸기
#### 승급
* 보통 메소드를 파라미터와 결과값을 Option이 감싸도록 승급(lift)가능하다
```
def lift[A,B](f: A => B): Option[A] => Option[B] = _ map f
```

예를 들면

```
val absWrappedOption: Option[Double] => Option[Double] = lift(math.abs)
```
예외가 발생할때, None으로 돌려준다. 즉 예외처리를 따로 하지 않아도 된다.

#### 감싸기 
* Try 함수를 구현해서 사용하자.
```
def Try[A](a: => A): Option[A] = try Some(a) catch { case e: Exception => None } 
```
* 게으른 파라미터: => A 로 표현함. 평가 시점에만 예외를 발생시키기 위해 이런 처리를 하였음.
* 예
```
// Try(age.toInt) 로 표현 가능. 파라미터 1개를 받는 함수는 대괄호로 ()를 대신 가능함
val optAge: Option[Int] = Try { age.toInt }  
```


### 4.3 문제 풀이 
#### 문제
* 두 Option값을 이항함수를 이용해서 결합하는 함수 map2를 만들어라. 
* lift나 Try를 사용할 필욘 없다.
```
def map2[A, B, C](a: Option[A], b: Option[B])(f: (A, B) => C): Option[C]
```
#### 풀이
```
object Option {
  def Try[A](a: => A): Option[A] = try Some(a) catch { case e: Exception => None }
  // 4.3
  def map2[A, B, C](a: Option[A], b: Option[B])(f: (A, B) => C): Option[C] = a.flatMap { x => b.map { y => f(x, y) } }
}
```
* 테스트
```
  test("should return results of map2") {
    println(Option.map2(Some(5), Some(10)) { (a: Int, b: Int) => a + b })
    println(Option.map2(None, Some(10)) { (a: Int, b: Int) => a + b })
    println(Option.map2(None, None) { (a: Int, b: Int) => a + b })
    println(Option.map2(Some(5), None) { (a: Int, b: Int) => a + b })
  }
```

### 4.4 문제풀이
Option의 목록을 받고 그 목록의 Some값을 담은 Option을 돌려주는 함수를 만들어라
```
def sequence[A](a: List[Option[A]]): Option[List[A]]
```

#### 풀이 
* List 의 문제라 match를 도입했다. 
* Option에서None처리를 위해선 flatMap 호출해야함.
```
  // 4.4 it's hard. 
  def sequence[A](a: List[Option[A]]): Option[List[A]] = a match {
    case Nil => Some(Nil)
    case h :: t => h.flatMap { x => sequence(t).map( x :: _ ) }
  }

```

### 4.5 문제 풀이
#### 문제 
```
  def traverse[A, B](a: List[A])(f: A => Option[B]): Option[List[B]] 
```
List요소를 한번씩 훑으면서 f를 처리하는 함수를 구현하라. 

#### 풀이

```
  // 4.5 
  def traverse[A, B](a: List[A])(f: A => Option[B]): Option[List[B]] = a match {
    case Nil    => Some(Nil)
    case h :: t => f(h).flatMap { x => traverse(t)(f).map(x :: _) }
  }
  
  def traverseViaMap[A, B](a: List[A])(f: A => Option[B]): Option[List[B]] = a match {
    case Nil => Some(Nil)
    case h :: t => map2(f(h), traverseViaMap(t)(f))(_ :: _)
  }

```

### For-Comprehension

스칼라는 승급함수를 자주 쓰기 때문에 for-Comprehension 구문을 제공한다.
이 구문은 flatMap, map 호출로 풀어버린다. 

map2의 예제

```
  def map2[A, B, C](a: Option[A], b: Option[B])(f: (A, B) => C): Option[C] =
  	 a.flatMap { x => b.map { y => f(x, y) } }
```

동일한 코드를 for-Comprehension을 사용한 것이다.

```
  def mapViaForComprehension[A, B, C](a: Option[A], b: Option[B])(f: (A, B) => C): Option[C] =
    for {
      aa <- a
      bb <- b
    } yield f(aa, bb)

```
for 중괄호 안에 bindings(aa <- a)가 있고 yield 다음엔 <- 묶음 좌변을 사용할 수 있다. 
flatMap으로 호출을 전개하고 마지막 묶음과 yield는 map으로 호출로 변환한다.


### Either 자료 형식

Option의 큰 단점은 예외가 발생했을때 무엇이 잘못되었는지 모른다.

원인을 추적하고 싶을땐 다른 자료형식이 필요하다. 여기선 스칼라 라이브러리에 있는 Either 자료형식을 구현해본다.


```
sealed trait Either[+E, +A]
case class Left[+E](value: E) extends Either[E, Nothing]
case class Right[+A](value: A) extends Either[Nothing, A]
```
Option과 유사하게 case가 두개 뿐이다. 하지만 둘다 값을 가진다는 차이가 있다.

Right 생성자는 성공
Left 생성자는 실패. type 매개변수로는 E를 사용한다.

mean 을 Either로 구현해보자

```
def mean(xs: IndexedSeq[Double]): Either[String, Dobule] = 
  if (xs.isEmpty)
    Left("Mean of empty list!")
  else
    Right(xs.sum / xs.length)
```

예외가 발생했다면 Left에 예외를 돌려주면 된다.

```
def safeDiv(x: Int, y:Int): Either[Exception, Int] = 
  try Right(x / y)
  catch { case e: Exception => Left(e) }
```

```
def Try[A](a: A): Either[Exception, A] = 
  try Right(a)
  catch { case e: Exception => Left(e)}
```


### 4.6 문제풀이 
#### 문제
```
 trait Either[+E, +A] {
  def map[B](f: A => B): Either[E, B]
  def flatMap[EE >: E, B] (f:A => Either[EE, B]): Either[EE, B]
  def orElse[EE >: E, B >: A] (b: Either[EE, B]): Either[EE, B]
  def map2[EE >: E, B, C](b: Either[EE, B])(f: (A, B) => C): Either[EE, C]
}
```
##### 풀이 

```
sealed trait Either[+E, +A] {
  def map[B](f: A => B): Either[E, B] = this match {
    case Right(a) => Right(f(a))
    case Left(e)  => Left(e)
  }

  def flatMap[EE >: E, B](f: A => Either[EE, B]): Either[EE, B] = this match {
    case Right(r) => f(r)
    case Left(e)  => Left(e)
  }
  def orElse[EE >: E, B >: A](b: Either[EE, B]): Either[EE, B] = this match {
    case Left(_)  => b
    case Right(r) => Right(r)
  }
  def map2[EE >: E, B, C](b: Either[EE, B])(f: (A, B) => C): Either[EE, C] =
    for {
      a <- this
      b1 <- b
    } yield f(a, b1)
}
```


* 이런 정의가 있으면 Either를 for-comprehension에 사용할 수 있다.(왜냐면 for-comprehension이 flatMap, map을 사용해서 수행하기 때문이다.)


### 4.7 문제풀이 
#### 문제
* 아래 함수를 구현해라. 발생한 첫 오류를 돌려주어야 한다.
```
def sequence[E, A](es: List[Either[E,A]]): Either[E, List[A]]

def traverse[E, A, B](as: List[A])(f: A => Either[E, B]): Either[E, List[B]]
```

#### 풀이

```
  // 4.7
  def sequence[E, A](es: List[Either[E, A]]): Either[E, List[A]] = es match {
    case Nil    => Right(Nil)
    case h :: t => h.flatMap { x => sequence(t).map { x :: _ } }
  }

  def traverse[E, A, B](as: List[A])(f: A => Either[E, B]): Either[E, List[B]] = as match {
    case Nil    => Right(Nil)
    case h :: t => f(h).flatMap { x => traverse(t)(f).map { x :: _ } }
  }

```

## 5장 엄격성과 나태성

스칼라의 라이브러리를 보면 큰 작업을 여러개의 작은 작업으로 나눈다
그 다음 작은 작업을 하나 완전히 수행하고 다음작업을 수행한다.

예를 들면

```
List(1,2,3,4).map(_ + 10 ).filter(_ % 2 == 0).map(_ * 3)
```
은 각 요소에 10을 더한 다음
짝수를 제거하고 
그 뒤에 3을 곱한다.


각 변환은 새 목록을 생성한다. 그 다음 변환에 입력으로만 쓰이고 폐기한다.
while로 구현할수도 있지만 고수준 합성 스타일 우지하면서 통합이 자동으로 이뤄지도록 하는게 이상적이다.
이런 비엄격성(나태성)을 이용하면 자동화된 루프 이용이 가능해짐. 

### 5,1 엄격한 함수 엄격하지 않은 함수
엄격한 함수: 항상 파라미터를 평가한다.
대부분의 프로그래밍 언어가 엄격한 함수를 지원한다. 
예) 
```
  def square(x: Double): Double = x * x
```

비엄격 함수: 함수가 하나 이상의 파라미터를 평가 히지않을 수도 있다.(평가하기도 한다)
* 비 엄격의 표현의 예로 && 와 Boolean을 생각할 수 있다.
* if도 함수라 생각하면 비엄격 부류다.
```
val result = if (input.isEmpty) sys.error("empty input") else input
```
좀 더 깔끔한 구문을 만들어보면
```
def if2(cond:Boolena, onTrue: () => A, onFalse: () => A): A = 
  if(conf) onTrue() else onFalse()

// 예제
if2( a< 22 , () => println("a"), () => println("b"))  
```
*  () => println("a")는 () => A를 생성하는 함수 리터럴이다. (() => A는 Function0[A] 형식)

### Thunk () => 
* 파라미터 중 평가를 미루는 경우는 Type바로 앞에 () => 를 넣어준다
* 표현식을 바로 평가하지 않는 것을 성크thunk 라고 한다.
* thunk는 메소드에서 참조할때 마다 호출한다
* () => A는 함수다.(파라미터는 없고 리턴값이 A인 함수)
* a: () => A 이면 a는 바로 평가하지 않음.
* a: () => A 일때 a()이면 강제로 평가함.

예를 들어

```
def maybeTwice(b: Boolean, i: => Int ) = if(b) i + i else 0

val x = maybeTwice(true, { println("hi"); 1 + 41})
// hi
// hi
// 82

```

캐싱을 적용해서 한번만 평가하려면? lazy 키워드를 사용해라

```
def maybeTwice2(b: Boolean, i: => Int) = {
	lazy val j = i
	if (b) j + j else 0
}

```

비엄격 함수의 인수는 이름으로 전달한다(call by name)

* 엄격성의 정의 : 표현식의 평가가 무한히 실해되면 한정한 값을 돌려주지 않고 오류를 던진다면 종료되지 않는  표현식 혹은 바닥 표현식이라고 한다.
만일 바닥으로 평가하는 모든 x에 대해 표현식 f(x)가 바닥이면 그 함수는 엄격한 함수다.

## 5.2 게으른 List(=Stream)

```
sealed trait Stream[+A] {
  def headOption: Option[A] = this match {
		case Empty => None
		case Cons(h, t) => Some(h())
	}
}
case object Empty extends Stream[Nothing]
case class Cons[+A] (h: () => A, t: () => Stream[A]) extends Stream[A]

object Stream {
	def cons[A] (hd: => A, tl: => Stream[A]): Stream[A] = {
		lazy val head = hd
		lazy val tail = tl
		Cons(()=> head, () => tail)
	}
	def empty[A]: Stream[A] = Empty

	def apply[A](as: A*): Stream[A] = 
		if (as.isEmpty) empty else cons(as.head, apply(as.tail: _*))
}
```

### 스트림 메모화를 통한 재계산 피하기
일반 생성자를 호출하면 expensive를 두번 호출한다.
```
val x = Cons(() => expensive(x), tl)
val h1 = x.headOption
val h2 = x.headOption
```

### smart constructer

* 위의 예제에서 Stream.cons  가 그 예제. 
* lazy 를 이용해서 함수 결과를 캐싱한다.
* apply 에보면 성크로 감쌌기 때문에 head 값이 필요하기 전까지 평가하지 않는다.


### 5.1 연습문제
#### 문제 
* Stream trait 내부에 toList를 만들어라

#### 풀이
* 내부 함수로 푸는 형태
* recursive하게 푸는 형태
* Tail Recursion을 이용하는 형태가 있다.
* Thunk가 있기 때문에(() => A) 강제로 평가 시킬 필요가 있다.
```
  def toListRecursive: List[A] = {
    def go(l: Stream[A]): List[A] = l match {
      case Cons(h, t) => h() :: go(t())
      case _          => List()
    }
    go(this)
  }
  def toListRecursive2: List[A] = this match {
    case Cons(h, t) => h() :: t().toListRecursive2
    case _          => List()
  }
  def toList: List[A] = {
    @tailrec
    def go(l: Stream[A], acc: List[A]): List[A] = l match {
      case Cons(h, t) => go(t(), h() :: acc)
      case _          => acc
    }
    go(this, List()).reverse
  }
```
### 5.2 연습문제
#### 문제 
* take, drop을 구현해라

#### 풀이
* 스마트 constructor를 이용했다. 
* 역시나 강제로 값을 평기해야 한다.
```
  def take(n: Int): Stream[A] = this match {
    case Cons(h, t) if n > 1  => Stream.cons(h(), t().take(n - 1))
    case Cons(h, _) if n == 1 => Stream.cons(h(), Empty)
    case _                    => Empty
  }

  def drop(n: Int): Stream[A] = {
    @tailrec
    def go(s: Stream[A], n: Int): Stream[A] = s match {
      case Cons(h, t) if n > 0 => go(t(), n - 1)
      case _                   => s
    }
    go(this, n)
  }

```
### 5.3 연습문제
#### 문제 
* takeWhile을 구현해라

#### 풀이

```
  def takeWhile(p: A => Boolean): Stream[A] = this match {
    case Cons(h, t) if p(h()) => Stream.cons(h(), t().takeWhile(p))   
    case _ => Empty
  }
```

### 5.3 프로그램 서술과 평가의 분리

* 관심사의 분리
계산의 서술과 실제 실행과 분리가 권장됨.
ex) 1급 함수는 일부 계산을 함수 내부에 가지고 있지만 계산은 파라미터가 전달되어야만 실행한다.

나태성을 통해 표현식 서술을 그 표현식의 평가와 분리가 가능하다

Stream.exists 예제

```
  def exists(p: A => Boolean): Boolean = this match {
    case Cons(h, t) => p(h()) || t().exists(p)
    case _          => false
  }

```

|| 는 두번째 파라미터에 엄격하지 않다. 
p(h()) 가 true이면 스트림을 더 훑지 않는다. 즉 tail로 exists를 실제로 수행하지 않는다.
위에선 명시적 재귀를 사용해서 구현했지만 exists는 foldRight로도 구현할 수 있다. 단 laziness 이다.

우선은 foldRight부터 보자.
```
  def foldRight[B](z: => B)(f: (A, => B) => B): B = this match {
    case Cons(h, t) => f(h(), t().foldRight(z)(f))
    case _          => z
  }

  def existViaFoldRight(p: A => Boolean): Boolean =
    foldRight(false)((a, b) => p(a) || b)

```
f가 둘째 파라미터를 평가에 엄격하지 않다.  f가 둘째 파라미터를 평가하지 않기로 했다면 순회가 일찍 종료된다.

이제 existViaFoldRight 설명이다. b는 스트림 tail을 fold하는 단계이지만 평가되지 않을 수 있다. p(a)가 true 라면 
b는 평가되지 않는다.
(단 스택 safe하진 않다.)


#### 연습문제
##### 문제 5.4
* forAll을 구현해라 
* 단, 중간에 조건을 만족하면 순회하지 말아야 한다.
```
 def forAll(p: A => Boolean): Boolean
```
##### 풀이
* || 를 이용해서 뒷 연산을 하지 않도록 처리 했다.
* existViaFoldRight 과 구현이 거의 동일하다.
```
  def forAll(p: A => Boolean): Boolean = this match {
    case Cons(h, t) => p(h()) || t().forAll(p)
    case _          => false
  }
  def forAllViafoldRight (p: A => Boolean): Boolean = 
  	foldRight(false)((a, b) => p(a) || b )
```
#### 문제 5.5
* foldRight을 이용해서 takeWhile을 구현해라
##### 풀이
```
  // 5.5. 
  def takeWhileViaFoldRight(p: A => Boolean): Stream[A] =
    foldRight(Stream[A]())((a, b) => if (p(a)) Stream.cons(a, b) else Empty)
```

#### 문제 5.6
* foldRight을 이용해서 headOption을 구현해라 

####  풀이 
* 패턴매칭과 foldRight으로 구현함
* Option의 foldRight 특성을 이해해야 정확히 풀수 있다.
```
  def headOptionViaFoldRight: Option[A] =
    foldRight(None: Option[A])((a, b) => b match {
      case None    => Some(a)
      case Some(h) => Some(h)
    })
  def headOption2ViaFoldRight: Option[A] =
    foldRight(None: Option[A])((h, _) => Some(h))
```

#### 문제 5.7
* foldRight을 이용해서 map, filter, append, flatMap을 구현해라. 단 append는 non-strict 하지 않게 해라
* def mapViaFoldRight filterViaFoldRight appendViaFoldRight flatMapViaFoldRight

####  풀이 
```
  def mapViaFoldRight[B](f: A => B): Stream[B] =
    foldRight(Stream[B]())((a, b) => Stream.cons(f(a), b))

  def filterViaFoldRight(f: A => Boolean): Stream[A] =
    foldRight(Stream[A]())((a, b) => if (f(a)) Stream.cons(a, b) else b)

  def appendViaFoldRight[B >: A](s: => Stream[B]): Stream[B] =
    foldRight(s)((a, b) => Stream.cons(a, b))

  def flatMapViaFoldRight[B](f: A => Stream[B]): Stream[B] =
    foldRight(Empty: Stream[B])((a, b) => f(a) appendViaFoldRight b)

}
```



## 
* 연습문제의 Stream의 메소드 구현은 점진적이다. 전체 결과를 생성하지 않고 참조 시점에 실제 계산을 한다
```
Stream(1,2,3,4).map( _ + 10 ).filter( _ % 2 == 0)
```
### 추적과정
```
Stream(1,2,3,4).map( _ + 10 ).filter( _ % 2 == 0).toList
cons(11, Stream(2,3,4).map(_ + 10)).filter( _ % 2 == 0).toList
Stream(2,3,4).map( _ + 10 ).filter( _ % 2 == 0).toList
cons(12, Stream(3,4).map( _ + 10 )).filter( _ % 2 == 0).toList
12 :: cons(13, Stream(4).map( _ + 10 )).filter( _ % 2 == 0).toList
... 중략

 12 :: 14 :: List()
```
* map과 filter 변환이 엇갈리는 방식이다.
* map등 중간 스트림이 완전히 인스턴스를 만들지 않는다. 
* 이를 일급 루프 라고 부르는 사람도 있다.
* 중간 스트림이 필요이상으로 데이터를 처리하지 않기 때문에 이런 combiner를 독창적인 방식으로 사용하는 경우가 있다

```
def find(p: A => Boolean): Option[A] =
  filter(p).headOption
```
* 메모리 사용에도 효율적이다. 딱 필요한 만큼만 메모리를 쓰고 나머지는 GC처리 할 수 있도록 만든다.

## 무한 스트림과 공재귀

```
val ones : Stream[Int] = Stream.cons(1, ones)
```
위 코드는 무한히 나열하는 stream 예제이다. 자기 자신을 참조하는 방식으로 만들어져 있다.
다만 지금까지 작성한 함수는 필요한 만큼만 결과를 출력한다
```
ones.take(5).toList
List[Int] = List(1,1,1,1,1)

ones.exists( _ % 2 != 0)
Boolean = true

ones.map(_ + 1).exists( _ % 2 == 0)
ones.takeWhile( _ == 1 )
ones.forAll( _ != 1)
```

### 5.8 문제풀이
* 무한 스트림을 만드는 메소드를 만들어라
```
def constant[A](a: A): Stream[A]
```
#### 풀이
```
  def constant[A](a: A): Stream[A] = {
    lazy val tail: Stream[A] = Cons(() => a, () => tail)
    tail
  }

  def constantViaSmartcons[A](a: A): Stream[A] =
    cons(a, constantViaSmartcons(a))

```

### 5.8 문제풀이
* 무한 스트림을 만드는 메소드를 만들어라
```
def constant[A](a: A): Stream[A]
```
#### 풀이
* constant가 좀더 효율적이다. 자기 자신 한개만 참조하도록 만들어져 있다.
```
  def constant[A](a: A): Stream[A] = {
    lazy val tail: Stream[A] = Cons(() => a, () => tail)
    tail
  }

  def constantViaSmartcons[A](a: A): Stream[A] =
    cons(a, constantViaSmartcons(a))

```
### 5.9 문제풀이
* n부터 1씩 증가하는 무한 스트림을 만들어라
```
def from(n: Int): Stream[Int]
```
#### 풀이
```
def from(n: Int): Stream[Int] = cons(n, from(n + 1))
```
### 5.10 문제
* 피보나치수열을 만드는 메소드를 만들어라
#### 풀이
```
  val fibs = {
    def go(f0: Int, f1: Int): Stream[Int] = {
      lazy val tail: Stream[Int] = cons(f1, go(f0, f0 + f1))
      tail
    }
    go(0, 1)
  }
```
### 5.11
* 일반화시킨 무한 스트림을 만드는 함수를 만들어라
```
def unfold[A, S](z: S)(f: S => Option[(A, S)]): Stream[A]
```
#### 풀이 
```
  def unfold[A, S](z: S)(f: S => Option[(A, S)]): Stream[A] =
    f(z) match {
      case Some((a, s)) => cons(a, unfold(s)(f))
      case None         => Empty
    }

```

* unfold는 공재귀(corecursive)함수다. 
* 재귀는 자료를 소비하지만 공재귀는 자료를 생산한다.
* 공재귀는 생산성이 유지되는 한 종료하지 않아도 된다.
* 공재귀를 보호되는 재귀(guarded recursion, 생산성을 공종료(cotermination) 라고 부른다.

### 5.12 연습문제
#### 문제 
* unfold를 이용해서 fibs, constant, ones, from을 구현해라

#### 풀이
```
  val fibsViaUnfold = unfold[Int, (Int, Int)]((0, 1)) { s => Some((s._1, (s._2, s._1 + s._2))) }

  def fromViaUnfold(n: Int): Stream[Int] = unfold(n) { s => Some((s, s + 1)) }

  def constantViaUnfold[A](a: A): Stream[A] = unfold(a) { s => Some((s, s)) }

  val onesViaViaUnfold = unfold(1) { s => Some((1, 1)) }
```

### 5.13 연습문제
#### 문제 
* unfold를 이용해서 mapViaUnfold, takeViaUnfold, talkeWhileViaUnfold, zipWith, zipAll을 구현해라

#### 풀이
* 패턴 매칭과 unfold속성을 이용해서 풀면된다.
```
  def mapViaUnfold[B](f: A => B): Stream[B] = Stream.unfold(this)(s => s match {
    case Cons(h, t) => Some((f(h()), t()))
    case _          => None
  })
  def takeViaUnfold(n: Int): Stream[A] = Stream.unfold((this, n))({
    case (Cons(h, t), 1)            => Some(h(), (Empty, 0))
    case (Cons(h, t), num) if n > 1 => Some((h(), (t(), num - 1)))
    case _                          => None
  })

  def talkeWhileViaUnfold(p: A => Boolean): Stream[A] = Stream.unfold(this) {
    case Cons(h, t) if p(h()) => Some((h(), t()))
    case _                    => None
  }
  def zipWith[B, C](r: Stream[B])(f: (A, B) => C): Stream[C] = Stream.unfold((this, r)) {
    case (Cons(h, t), Cons(h2, t2)) => Some(f(h(), h2()), (t(), t2()))
    case _                          => None
  }
  def zipAll[B](s2: Stream[B]): Stream[(Option[A], Option[B])] = Stream.unfold((this, s2)) {
    case (Cons(h, t), Cons(h2, t2)) => Some((Some(h()), Some(h2())), (t(), t2()))
    case (Cons(h, t), Empty)        => Some((Some(h()), None), (t(), Empty))
    case (Empty, Cons(h2, t2))      => Some((None, Some(h2())), (Empty, t2()))
    case (Empty, Empty)             => None
  }
```
### 5.14 연습문제
#### 문제 
* unfold를 이용해서 mapViaUnfold, takeViaUnfold, talkeWhileViaUnfold, zipWith, zipAll을 구현해라
```
 def startsWith[A](s: Stream[A]): Boolean
```
#### 풀이
* zipAll, takeWhile, forAll을 이용해서 풀면된다.
* 구현한 함수를 이용해서 합성으로 문제풀이를 하면된다.
* `fc(x) = f(c(x))` 형태로 문제를 풀자.
```
  def startsWith[A](s: Stream[A]): Boolean = zipAll(s).takeWhile( a => !a._2.isEmpty ).forAll{
    case (h1,h2) => h1 == h2
  }
```
### 5.15 연습문제
#### 문제 
* unfold를 이용해서 tails를 만들자.
* `Stream(1,2,3).tails == Stream(Stream(1,2,3), Stream(2,3), Stream(3))`
```
def tails: Stream[Stream[A]]
```
#### 풀이
```
  def tails: Stream[Stream[A]] = Stream.unfold(this){
    case Empty => None
    case s => Some((s, s drop 1)) 
  } appendViaFoldRight Stream(Empty)
```

### 5.16 연습문제
#### 문제

* tails를 일반화한 scanRight함수를 작성하라. 이 함수는 중간결과의 스트림을 돌려주는 foldRight다.
예로 
```
Stream(1, 2, 3).scanRight(0)(_ + _).toList
List(6,5,3,0)
List(1+2+3+0, 2+3+0, 3+0, 0) // 이 표현과 동등
```
* unfold를 못쓰는 이유는? 
#### 풀이
* unfold는 좌에서 우로 stream을 생성하기 때문에 쓸 수 없다.
* foldRight를 이용해서 풀 수 있다. 
```
 def scanRight[B](z: B)(f: (A, => B) => B): Stream[B] =
    foldRight((z, Stream(z)))((a, b) => {
      lazy val b2 = b
      val r = f(a, b2._1)
      (r, Stream.cons(r, b2._2))
    })._2
```
## 6. 순수 함수적 상태

* 상태를 다루는 순수 함수적 프로그래밍을 다룬다.
* 예) 난수 발생
* `scala.util.Random`클래스를 사용한다.

```

scala> val rng = new scala.util.Random
scala> rng.nextDouble

res0: Double = 0.8277158914379049

scala> rng.nextInt(10)
res1: Int = 5

```

+ rng의 메소드는 내부적으로 상태를 갖는다. 
   - 상태가 없다면 똑같은 값이 리턴된다.
+ 테스트성에 맞춰서 RNG를 보자. 
   - 매번 상태가 파괴된다. 즉 재현이 안된다.
   - 인수로 RNG를 전달 하는 것도 방법이지만, 내외부 상태 추적을 해야한다.
   - random이 내부에서 몇번 호출되었는가 정보도 필요하다.
   ```
   def rollDie(rng: scala.util.Random): Int = rng.nextInt(6)
   ```
   - side-effect 제거가 가장 좋은 해결책이다.

###  순수 함수적 난수 발생
* 참조 투명성을 보장하려면 상태 갱신을 명시적으로 드러내자.
```
trait RNG {
  def nextInt: (Int, RNG)
}
```
* `nextInt`는 발생한 난수만 돌려주고, 내부 상태는 제자리 변이를 통해서 갱신한다.
* 그리고 이 트레잇은 난수와 새 상태를 돌려주며 기존 상태는 유지한다.
* 다음 상태를 계산하는 관심사와 새 상태를 프로그램에 알려주는 관심사를 분리한다.
* 다음은 이 구현 예제다
```
case class SimpleRNG(seed: Long) extends RNG {
 def nextInt: (Int, RNG) = {
   val newSeed = (seed * 0x5DEECE66DL + 0xBL) & 0xFFFFFFFFFFFFFFL
   val nextRNG = SimpleRNG(newSeed)
   val n = (newSeed >>> 16).toInt
   (n, nextRNG)
 }
}
// 실행 결과
scala> val rng = SimpleRNG(42)
rng: SimpleRNG = SimpleRNG(42)

scala> val (n1, rng2) = rng.nextInt
n1: Int = 16159453
rng2: RNG = SimpleRNG(1059025964525)

scala> val (n2, rng3) = rng2.nextInt
n2: Int = -1281479697
rng3: RNG = SimpleRNG(62684936753093620)
```
* rng를 호출하면 늘 같은 결과가 나온다. 즉, 이 함수는 순수하다. 

### 상태 있는 API를 순수하게 만들기
+ 동일하게 이런  패턴을 이용하면 순수하게 만들 수 있다.
+ 이 패턴은 계산한 다음상태를 전달하는 것을 호출자에게 맡긴다.
+ 순수함수를 이용해서 다음 상태를 갱신하면 효율성 손실이 발생할 수 있다.
  - 바로 제자리 갱신이 안되고 복사를 한뒤 갱신하므로..
  
* 예

```
class Foo {
  private var s: FooState=  ... 
  def bar: Bar
  def baz: Int
}
```
위의 예제를 아래처럼 순수 함수형태로 바꾸는게 가능하다

```
trait Foo {
  def bar: (Bar, Foo)
  def baz: (Int, Foo)
}
```

### 문제풀이 6.1
RNG.nextInt를 이용해서 0이상 `Int.MaxValue` 이하의 난 수 정수를 생성하는 함수를 작성하라. 
`def nonNegativeInt(rng: RNG): (Int, RNG)`
#### 풀이
* `-` 처리를 하면 역수를 구할 수 있다. Int는 음수의 경우 양수보다 범위 커버가 크므로 +1 을 해준다.
```
  def nonNegativeInt(rng: RNG): (Int, RNG) = {
    val (i, r) = rng.nextInt
    (if (i < 0) -(i + 1) else i, r)
  }
```
### 문제풀이 6.2
0이상 1 미만의 double 난수를 발생하는 메소드를 만들어라

#### 풀이
* 랜덤값이 Int.MaxValue가 나올 수 있다. 그럼 랜덤값 나누기 int 최대값을 해보면 1이 나온다 그래서 분모에 +1을 해서 이를 방지한다.
```
  def double(rng: RNG): (Double, RNG) = {
    val (i, r) = nonNegativeInt(rng)
    (i.toDouble / Int.MaxValue.toDouble + 1, r)
  }
```

### 문제풀이 6.3
각 난수쌍을 만드는 함수를 작성하라, 가능하면 만든 메소드를 재사용해라
```
 def intDouble(rng: RNG): ((Int, Double), RNG)
 def doubleInt(rng: RNG): ((Double, Int), RNG)
 def double3(rng: RNG): ((Double, Double, Double), RNG)
```
#### 풀이
```
  def intDouble(rng: RNG): ((Int, Double), RNG) = {
    val (i, rng2) = rng.nextInt
    val (d, rng3) = double(rng)
    ((i, d), rng3)
  }

  def doubleInt(rng: RNG): ((Double, Int), RNG) = {
    val ((i, d), rng2) = intDouble(rng)
    ((d, i), rng2)
  }

  def double3(rng: RNG): ((Double, Double, Double), RNG) = {
    val (d1, r1) = double(rng)
    val (d2, r2) = double(r1)
    val (d3, r3) = double(r2)
    ((d1, d2, d3), r3)
  }
```

### 문제풀이 6.4
정수 난수 목록을 만드는 함수를 만들어라
#### 풀이
```
  def ints(count: Int)(rng: RNG): (List[Int], RNG) = {
    @tailrec
    def gen(n: Int, r: RNG, l: List[Int]): (List[Int], RNG) =
      if (n == 0) (l, r)
      else {
        val (i, nr) = r.nextInt
        gen(n - 1, nr, i :: l)
      }
    gen(count, rng, List())
  }
```

### 6.4 상태 동작을 위한 API 개선
앞의 함수를 살펴보자.
매번 RNG => (A, RNG) 형식을 공통적으로 사용한다.
이 말은 RNG를 새 RNG 상태로 변환한다는 것이다. 
이런 함수를 상태 동작(stat action) 이나 상태전이(state transition)이라 부른다.
이런 함수는 combinator를로 조합할 수 있다. 

우선 상태 동작 data type의 alias 를 만들자. 
`type Rand[+A] = RNG => (A, RNG)` 

이걸 만드는 이유는 호출자가 매번 상태를 전달하는 반복적이고 지루한 작업이기 때문이다. 그래서 조합기가 자동으로 넘겨주도록 한다.


```
  val int: Rand[Int] = _.nextInt
```
+ `Rand[Int]` 값은 
  - 한 상태동작에 의존한다
  - 이 상태를 이용 A를 생성. 
  - 새로운 상태로 전이 한다.

+ 이렇게 combinator를 만들다보면 모든 전달을 자동으로 처리하는 DSL을 만들 수 있다.


예로 RNG를 사용하지 않고 그대로 전달하는 unit 동작이다. 
```
  def unit[A](a: A): Rand[A] = rng => (a, rng)
```

그리고 한 상태 동작의 출력을 변환하지만 상태는 수정하지 않는 map도 생각할 수 있다.

```
def map[A,B](s: Rand[A])(f: A => B): Rand[B] =
  rng => {
    val (a, rng2) = s(rng)
    (f(a), rng2)
  }
```

위의 내용을 응용해서 0보다 크고 2로 나누어 떨어지는 Int 발생 함수를 만들 수 있다.
```
 def nonNegativeEven: Rand[Int] = map(nonNegativeInt)(i => i - i % 2)
```

#### 6.5 연습문제 
6.2의 double을 map을 사용해서 우아하게 풀어봐라.
##### 풀이 
```
  def doubleViaMap: Rand[Double] = 
    map(nonNegativeInt)(i => i.toDouble / (Int.MaxValue.toDouble + 1))
```

#### 상태 동작들의 조합

* 위에 다룬 예제는 intDouble, DoubleInt를 다룰정도로 강력하진 않다 RNG를 이항함수로 조합하는 새로운 map2가 필요하다.

#### 6.6 연습문제 
* 두 상태동작 ra, rb와 이들의 결과를 조합하는 함수 f 를 받고 두 동작을 조합한 새동작을 돌려준다.
```
def map2[A,B,C](ra: Rand[A], rb: Rand[B])(f: (A, B) => C): Rand[C]
```
##### 풀이
* return 값은 (RNG => (A, RNG))이다.
* rng값은 공통이므로 계속 하나씩 하나씩 상태 전이를 하면 된다.
```
  def map2[A, B, C](ra: Rand[A], rb: Rand[B])(f: (A, B) => C): Rand[C] =
    rng => {
      val (a, rng2) = ra(rng)
      val (b, rng3) = rb(rng2)
      (f(a, b), rng3)
    }
```

* 위 map2를 이용하면 임의의 RNG 상태 동작을 조합할 수 있다.

```
def both[A,B](ra: Rand[A], rb: Rand[B]): Rand[(A,B)] =
  map2(ra, rb)((_, _))
```

intDouble과 dobuleInt를 손쉽게 구현할 수 있다.

```
val randIntDouble: Rand[(Int, Double)] = both(int, double)
val randDoubleInt: Rand[(Double, Int)] = both(double, int)

```

#### 6.7 연습문제 
* 상태전이 목록 전체를 조합하는게 가능해야한다. sequence를 구현해라
그리고 이 함수를 이용해서 ints를 다시 구현하라.
ints함수 구현에서 x가 n번 되풀이되는 목록을 만들려면 List.fill(n)(x)를 사용해라 
```
def sequence[A](fs: List[Rand[A]]): Rand[List[A]]
```
##### 풀이
* 처음 풀이는 map2 풀이를 참고해서 풀었다.
* 상태 정보가 들어나게 되므로 그다지 좋은 풀이는 아니다.
* 두번째는 List의 foldRight를 이용해서 풀었다.
```
  def sequence[A](fs: List[Rand[A]]): Rand[List[A]] = {
    rng =>
      {
        @tailrec
        def go(l: List[Rand[A]], r: RNG, rt: List[A]): (List[A], RNG) =
          l match {
            case h :: t => {
              val (s, nr) = h(rng)
              go(t, nr, s :: rt)
            }
            case Nil => (rt, r)
          }
        go(fs, rng, List())
      }
  def sequence2[A](fs: List[Rand[A]]): Rand[List[A]] =
    fs.foldRight(unit(List[A]()))((f, acc) => map2(f, acc)(_ :: _))

  def intsViaSequence(count: Int): Rand[List[Int]] =
    sequence2(List.fill(count)(int))
```


#### 내포된 상태 동작 

RNG를 명시적으로 언급하거나 전달하지 않아도 구현이 가능하다.
map과 map2로 작성이 어려운 것도 있다.

예제는 0이상 n미만 난수 발생 함수다 .

```
def nonNegativeLessThan(n: Int): Rand[Int]
= map(nonNegativeInt) { _ % n }
```
위처럼 구현하면 요구 범위 난ㅅ구가 만들어지지만, `Int.MaxValue`가 n으로 나누어 떨어지지 않을 수도 있다.
그래서 전체적으로 난수가 치우친다. 나머지보다 작은 수들이 자주 나타난다는 것이다. 
아래는 개선한 풀이다.

```
def nonNegativeLessThan(n: Int): Rand[Int] = 
  map(nonNegativeInt) { i => 
    val mod = i % n 
    if ( i + (n - 1) - mod >= 0 ) mod else nonNegativeLessThan(n)(???)
  }
```
* 위 구현은 nonNegativeLessThan(n) 형식이 자리가 맞지 않는다. 이 함수는 Rand[Int]를 리턴해야하고 RNG 하나를 인수로 받아야 한다.
* 그래서 map을 사용하지 않고 명시적으로 전달하는 방법이다.

```
def nonNegativeLessThan(n:Int): Rand[Int] = {
 rng => 
    val (i, rng2) = nonNegativeInt(rng)
    val mod = i % n 
    if (i + (n - 1) - mod >= 0 )
      (mod, rng2)
    else nonNegativeLessThan(n)(rng2)  

}
```

map 말고 이런 전달을 처리 해주는 조합기가 있으면 좋을 것이다.


#### 6.8 연습문제
* flatMap을 구현하고 이를 이용해서 nonNegativeLessThan을 구현해라
```
def flatMap[A, B](f: Rand[A])(g: A => Rand[B]): Rand[B]
```

##### 풀이
```
  def flatMap[A, B](f: Rand[A])(g: A => Rand[B]): Rand[B] =
    rng => {
      val (a, nr) = f(rng)
      g(a)(nr)
    }

  def nonNegativeLessThan(n: Int): Rand[Int] =
    flatMap(nonNegativeInt) { i =>
      val mod = i % n
      if (i + (n - 1) - mod >= 0) unit(mod) else nonNegativeLessThan(n)
    }
```

#### 6.9 연습문제
* flatMap을 이용해 map, map2를 구현해라

##### 풀이

```
  def mapViaFlatMap[A, B](s: Rand[A])(f: A => B): Rand[B] =
    flatMap(s)(a => unit(f(a)))

  def map2ViaFlatMap[A, B, C](ra: Rand[A], rb: Rand[B])(f: (A, B) => C): Rand[C] =
    flatMap(ra)(a => {
      flatMap(rb)(b => {
        unit(f(a, b))
      })
    })
```

* nonNegativeLessThan을 이용해서 rollDie를 구현해보자.

```
def roleDie: Rand[Int] = nonNegativeLessThan(6)
```
* 여전히 하나 모자라는 오류가 있다. 이 함수를 RNG 상태로 검사하면 함수 반환값이 0이 되는 RNG를 발견할 수 있다.
* 0이 나오는 경우는 아래처럼 수정하면 된다.

```
def roleDie: map(nonNegativeLessThan(6))(_ + 1)
```

### 6.5 일반적 상태 동작 자료 형식 

* unit, map, map2, flatMap, sequence은 범용함수다.
예를 들어 map은 RNG 상태 동작을 다루는지 상관않는다. 
그래서 아래처럼 일반적인 서명을 사용할 수 있다. 
```
def map[S, A, B](a: S => (A, S))(f: A => B): S => (B, S)
```
* 이렇게 서명을 바꿔도 map 구현은 수정할 필요가 없다. 
* 이제 좀더 범용적인 상태를 처리할 수 있는 형식을 생각해보자

```
type State[S, +A] = S => (A, S)
```
* 이 State는 상태 동작 상태 전이를 대표한다. statement를 대표한다고 볼수도 있다.
* 아래와 같은 case class 형식으로 만들 수도 있다. 
```
case class State[S, +A](run: S => (A, S))
```
위에 State를 이용해 Rand를 Alias로 만들 수 있다.

```
type Rand[A] = State[RNG, A]
```

#### 6.10 연습문제
* unit, map, map2, flatMap, sequence를 일반화하라. 가능하면 State case class 에 넣고 불가능하면 State Companion 객체에 넣자.

##### 풀이
* State case class에 있는 경우는 A에 대한 고려를 할 필요가 없다.( 같은 객체이기 대문에)
* sequence는 foldRight, foldLeft로 구현 할 수 있다. (foldLeft가 더 빠르다.)
```
case class State[S, +A](run: S => (A, S)) {
  // 6.10
  def flatMap[B](f: A => State[S, B]): State[S, B] =
    State(
      s => {
        val (a, s1) = run(s)
        f(a).run(s1)
      })
  def map[B](f: A => B): State[S, B] =
    flatMap(a => State.unit(f(a)))

  def map2[B, C](sb: State[S, B])(f: (A, B) => C): State[S, C] =
    flatMap(a => sb.map { b => f(a, b) })

}

object State {
  def unit[S, A](a: A): State[S, A] = State { (st: S) => (a, st) }

  def sequence[S, A](fs: List[State[S, A]]): State[S, List[A]] = {
    def go(s: S, l: List[State[S, A]], acc: List[A]): (List[A], S) = {
      l match {
        case Nil => (acc, s)
        case h :: t => h.run(s) match {
          case (a, s1) => go(s1, t, a :: acc)
        }
      }
    }
    State((s: S) => go(s, fs, List()))
  }
}
```
### 6.6 순수 함수적 명령식 프로그래밍
* 위에 작성해본 예제에서 상태동작을 실행하고 결과를 val에 배정한다.
* 또 그 val을 사용하는 다른 상태 동작을 실행하고 그 걸 다시 val에 배정하는 방식을 사용했다. 
* 명령식 프로그래밍과 비슷하다. 
* 하지만 실제는 상태동작 State이고, 이건 함수다. 인수를 받으면서 현재 프로그램 상태를 읽고, 
그냥 값을 돌려줌으로 써 프로그램 상태를 수정한다. 
* 위의 구현한 형태를 보면 명령식 프로그램 성격은 많이 사라졌다.

예) 

```
val ns: Rand[List[Int]] = 
  int.flatMap (x => 
    int.flatMap( y => 
      ints(x).map(xs => 
        xs.map(_ % y)
      )
    )
  )
```

위 코드는 이해 하기 어렵다. map과 flatMap이 연달아 나오면 쓸 수 있는 for-comprehension 을 이용해서 명령식 스타일로 만들 수 있다.

```
val ns: Rand[List[Int]] = for {
	x <- int
	y <- int
	xs <- ints(x)	
} yield xs.map(_ % y )
```


for-comprehension 을 이용하는 프로그래밍 방식 이용하려면 기본적은 State combinator 2개다.
현재 상태를 얻는 get, 새 상태를 설정하는 set이다. 공통 코드를 만들어보자

```
def modify[S](f: S => S): State[S, Unit] = for { 
  s <- get
  _ <- set(f(s))
} yield ()

def get[S] : State[S, S] = State(s => (s, s))
def set[S](s: S): State[S, Unit] = State(_ => ((), s))
```

이 두 간단한 동작과 이전에 작성한 combinator가 있으면 어떤 종류의 상태 기계나 상태가 있는 프로그램도 순수 함수적 방식으로 구현이 가능하다.

#### 6.11 연습문제
* 사탕 판매기 예제를 풀어라.
이 판매기에는 두가지 종류의 입력이 있다. 하나는 동전이고 또 하나는 상탕이 나오는 손잡이 이다. 
```
sealed trait Input
case object Coin extends Input
case object Turn extends Input

case class Machine(locked: Boolean, candies: Int, coins: Int)
```
* 사탕 판매기의 작동 규칙은 다음과 같다. 

+ 잠겨진 판매기에 동전을 넣으면 사탕이 남아 있는 경우 잠김이 풀린다.
+ 풀린 판매기의 손잡이를 돌리면 사탕이 나오고 판매기가 잠긴다.
+ 잠긴 판매기의 손잡이를 돌리거나 풀린 판매기에 동전을 넣으면 아무일도 생기지 않는다 
+ 사탕이 없는 판매기는 모든 입력을 무시한다.

* `simulateMachine` 메소드는 입력에 기초해서 판매기를 작동하고 작동이 끝난뒤에 판매기에 있는 동전 개수와 사탕 개수를 돌려줘야 한다.
예를 들어 동전이 10개 사탕이 5개 있는 Machine에 총 4개의 사탕이 성공적으로 팔렷다면 출력은 (14, 1)이어야 한다.

##### 풀이

```

sealed trait Input

case object Coin extends Input
case object Turn extends Input

case class Machine(locked: Boolean, candies: Int, coins: Int)

```

### 요약
+  전반적인 패턴은 다음과 같다.
  - 상태를 인수로 받는다.
  - 새 상태를 결과와 함께 돌려주는 순수 함수 사용 
+ 부수 효과에 의존하는 API를 만나면 리팩토링 해봐라.


## 7. 순수 함수적 병렬성

* 계산의 서술과 실행을 분리한다.
* parMap combinator를 구현한다. 
 - 함수 f를 모든 컬렉션 요소에 적용할 수 있다.
 ```
 	val outputList = parMap(inputList)(f)
 ```
* 라이브러리가 처리하고자 하는 용례를 만든다.
* 점점 복잡한 용례를 거치면서 도메인과 설계 공간 이해도를 높인다.
* 대수적 추론에 역점을 두고 API를 특정 법칙에 따르는 대수로 서술할 수 있다.
* 관례가 실용적이진 않다. 
* 라이브러리를 만들려면 기존 설계의 근본 가정을 고찰한뒤 다른 경로를 택하고, 문제 공간에서 다른 사람이 고찰하지 못한 것을 발견할 수 있다.
* 근본적으로 라이브러리가 side effect를 허용하지 않는 것이다.

### 7.1 자료형식과 함수의 선택
* "병렬 계산을 생성할 수 있어야 한다" 
  - 이 착안을 구현가능한 걸로 상세화 해보자.
  ```
  	def sum(ints: Seq[Int]): Int = 
  	  ints.foldLeft(0)((a, b) => a + b)
  ```
  - foldLeft는 표준 라입인 Seq에 있다.
  - 이 방법 말고 분할 정복 방법을 써보자
  ```
   def sum(ints: IndexedSeq[Int]): Int = 
     if (ints.size <= 1)
       ints.headOption getOrElse 0
     else {
       val (l, r ) = ints.splitAt(ints.length /  2)
       sum(l) + sum(r)
     }
  ```
  - 이 코드는 순차열 splitAt으로 두개로 분할해서 재귀적으로 부분합을 구한다.
  - 이 구현은 병렬화 할 수 있다. 두 절반을 병렬로 합할 수 있다.
  - 이 계산을 병렬화 하려면 어떤 종류의 자료형식과 함수가 필요할지 고민해보자
    - 이러면 이전 관점과 다른 관점으로 본다
    - 간단한 예제로부터 영감을 얻어서 이상적인 API 를 직접 설계 구현하는 방식을 선호한다.

#### 7.1.1 병렬 계산을 위한 자료형식 하나 
* `sum(l) + sum(r)`은 병렬 계산 자료 형식이 1개의 결과를 담아야 한다는걸 알 수 있다.
* 그 결과는 어떤 의미 있는 형식이어야 하며
* 추출함수도 필요하다.
* 지금 결과를 담을 컨테이너 형식을 창안하자(Par[A])

* `def unit[A](a => A): Par[A]` : 평가되지 않은 A를 받고 이걸 개별적은 쓰레드로에서 평가할 수 있는 계산을 돌려준다. unit인 이유는 쓰레드 한개가 이걸 감쌀 수 있다는 점이다.
* `def get[A](a: Par[A]): A` : 병렬 계산에서 결과값을 추출한다.

* 우선은 예제를 통해서 자료형식과 함수만 뽑아내자.

```
def sum(ints: IndexdedSeq[Int]): Int = 
  if (ints.size <=  1)
    ints headOption getOrElse 0
  else 
    val (l,r) = ints.splitAt(ints.length / 2)
    val sumL : Par[Int] = Par.unit(sum(l))
    val sumR : Par[Int] = Par.unit(sum(r))
    Par.get(sumL) + Par.get(sumR)
```
* 이제 unit, get의 의미를 선택하자.
* unit은 주어진 인수를 개별적인 쓰레드로 즉시 평가 or get 호출시 평가. 
 - 하지만 병렬 잇점을 취하려면 동시에 평가 시작후 즉시 반환해야 한다.
 - 스칼라는 왼쪽에서 오른쪽으로 엄격하게 평가한다. 즉 unit이 get이 호출될때까지 기다리면 첫번째 병렬 계산이 끝나야 다음 병렬 계산을 시작할 수 있다. 
* ` Par.unit(sum(l)) + Par.unit(sum(r))`은 unit을 즉시 평가하면 get 평가 완료를 기다린다
  + get은 unit 한정 부수 효과가 존재한다.

##### 다음의 과정을 거쳤다.
1. 간단한 예제 작성
2. 예제를 살피고 설계상의 선택 문제를 찾아냈다
3. 몇가지 실험을 통해서 특정 선택 사항의 흥미로운 결과를 발견했다.

#### 7.1.2 병렬 계산의 조합 

앞에서 말한 unit과 get의 조합을 어떻게피할까?

```
def sum(ints: IndexedSeq[Int]): Par[Int] = 
  if (ints.size <= 1)
    Par.unit(ints.headOption getOrElse 0 )
  else {
 	val (l,r) = ints.splitAt(ints.length / 2)
 	Par.map2(sum(l), sum(r))(_ + _)

  }
```
#### 연습문제 7.1
* Par.map2는 두 병렬 계산의 결과를 결합하는 고차 함수이다. 이 함수의 서명은 무엇일까?
일반적으로 작성해보라

```
def map2[A, B, C](a:Par[A], b: Par[B])(z: (A, B) => C): Par[C]
```

* 이제는 재귀에서 unit을 호출하지 않는 점을 주목하자. 이제는 unit 인수가 게으른 인수 여야 하는지 명확하지 않다. 
* map2는 계산에 동등한 기회를 주어 병렬로 실행될 수 있게 해야한다.
* 아래 평가의 TC를 보자


## 참고 자료 
* [스칼라 기본 타입](https://twitter.github.io/scala_school/ko/type-basics.html)
* [FP in Scala 답](https://github.com/fpinscala/fpinscala)
