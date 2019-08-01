---
title: HashCode() 와 equals()
categories:
- sookstory
excerpt: |
  예전에 JPA를 사용할 때 따라하던 예제들 중 저 hashcode와 equals 를 override한 부분이 있었다. equals는 자바를 처음 접했을 때 부터 비교를 위해 자주 사용하던 메소드이지만 hashcode는? 주소인가?(...) hash가 앞에 붙은 것들은 이미 여러번 본 적이 있다. hashmap, hashtable 등과 같은 자료구조에서. hashcode는 뭐고, 이 두개의 차이점은 뭐고 어떻게 쓰이는걸까?
feature_text: |
  ## HashCode() 와 equals() 
  HashCode() 와 equals()
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---


HashCode와 Equals
====

예전에 JPA를 사용할 때 따라하던 예제들 중 저 hashcode와 equals 를 override한 부분이 있었다. equals는 자바를 처음 접했을 때 부터 비교를 위해 자주 사용하던 메소드이지만 hashcode는? 주소인가?(...)  
hash가 앞에 붙은 것들은 이미 여러번 본 적이 있다. hashmap, hashtable 등과 같은 자료구조에서. hashcode는 뭐고, 이 두개의 차이점은 뭐고 어떻게 쓰이는걸까?   
  
equals()
----

내용이 같은 것을 비교하는 메소드이다. 라고 해서 임의의 String 값만을 가지는 Value라는 객체를 두개 만들어서 비교해보기로 했다.   

<pre><code>
public class Value {

	private String value;

	public String getValue() {
		return value;
	}

	public void setValue(String value) {
		this.value = value;
	}
}
</code></pre>
   
<pre><code>

		Value value1 = new Value();
		value1.setValue("value1");
		
		Value value2 = new Value();
		value2.setValue("value1");
		
		System.out.println("value1과 value2가 equals? " + value1.equals(value2));
</code></pre>
   
결과는 false였다..   
상위 Object 클래스의 equals()는 아래와 같이 구현되어있다.   
   
<pre><code>

    public boolean equals(Object obj) {
        return (this == obj);
    }
</code></pre>

이러면 당연히 안의 내용을 확인하는 것이 아니게 된다. 그래서 오버라이드해서 내용물을 비교하도록 짜주어야하는데, 우리가 처음에 == 와 equals의 차이점에 대해 배울 때 분명 equals는 내용물을 비교했다.     
예를 들어서 "".equals("") 이렇게! 그렇다면 String은 어떻게 구현되어있는 걸까?   

<pre><code>

    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
</code></pre>   
  
String으로 만약 형변환이 되면 if문에서 value값의 길이와 값을 비교한다. 그렇다. String은 == 만으로는 비교가 불가능해서 오버라이드해서 내용물을 비교하게끔 구현해준 것이다. 그러니까 값을 여러개 포함하는 오브젝트를  
구현할 땐 String 처럼 값을 전부 비교해주어야 equals를 정확하게 구현한 것이 된다. equals는 내용이 같음을 검사하는 메소드니까!  
  
hashCode()
----

객체가 같은 객체인지를 비교할 때 사용된다. 식별값이라고 생각하면 된다. hashCode()의 상위 코드는 아래와 같다.  
  
<pre><code>
public native int hashCode();
</code></pre>

hashcode를 만들어서 return 해주는 메소드인데, 위의 String이 같은 객체를 두개 만들어서 출력하면 당연하겠지만 다르다. 그러니까 equals()는 오버라이드해서 같은 값으로서 true로 나오게 하더라도,  
객체의 hashcode가 다른것이다.  
  
<pre><code>

		Value value1 = new Value();
		value1.setValue("value1");
		
		Value value2 = new Value();
		value2.setValue("value1");
		
		System.out.println(value1.hashCode() + " , " + value2.hashCode());
</code></pre>  
  
위의 결과는 205988608 , 1600427200 이렇게 나온다. String의 hashCode 메소드는   
  
<pre><code>

  public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
</code></pre>
   
이 로직으로 새로운 hash를 만들어서 리턴한다. 그래서 정말 동일한 객체임을 확인하려면 이 값도 같아야 하는 것이다. 그래서 HashTable이나 HashMap에서 값이 같으면 같은 객체로 사용하고 싶은 경우는  
이 메소드를 오버라이드해서 같은 해시값을 리턴하도록 구현해주면 된다.  


덧,  
  
위의 hashCode() 구현에서 31의 의미는? 소수이고 홀수이기 때문이다. 같은 값이 나올 확률이 낮기 때문이다. 소수이고 홀수인 숫자들은 많은데??   
'소수이고 홀수인 값들 중에 근처에 2의 승수로 올라가는 숫자가 있다' 가 지금까지 찾아본 내용들의 결론이다.(내 생각엔..) 그러니까 무슨말이냐면, 31 옆에 있는 32란 숫자가 2의 승수이고(2의 5승)  
이는 시프트 연산으로 쉽게 구현할 수 있다. 그래서 다른 숫자들보다 빠르게 연산할 수 있다.. 는 것이다.  

이를 설명하는 글에는 honor's method와 31N=32N-N 이라는 식이 계속 등장한다.  이 부분은 다음번에 정리해보도록 하자.

