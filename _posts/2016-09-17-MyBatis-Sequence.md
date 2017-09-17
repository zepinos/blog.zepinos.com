---
layout: post
title: MyBatis(iBatis) 에서 Oralce 등의 Sequence 을 쓸 때 &lt;selectKey&gt; 을 권장하는 이유
author: zepinos
comments: true
categories: Java-MyBatis
cover:  "/assets/instacode.png"
---

이 글을 쓰는 시점에서 약간 지난(2~3주 쯤 된 것 같다) 시점에서 한 커뮤니티에 대략 아래와 같은 글이 올라왔다.


> Oracle, MyBatis 을 쓰는데, 아래와 같이 insert 을 할 때 문제가 발생합니다.
~~~sql
insert into 테이블 (컬럼1, 컬럼2, ...) values (sequence.nextval, 값2, ...)
insert into 테이블 (컬럼1, 컬럼2, ...) values (sequence.**currval**, 값2, ...)
~~~


위와 같이 작성했는데 문제가 된다면 보통은 트랜젝션을 선언해주지 않아서 다른 세션으로 insert 가 실행되었을 가능성이 크다.

그런데, 애들 때문에 시달리는 와중에 모바일로 내용을 보고(애들 때문에 컴을 켤 엄두도 안난다)...세션 생각은 덜 한 상태에서, 조금 틀린 내용을 포함한 답을 해줬다. 질문자가 이미 트랜젝션을 해주니 문제가 해결되었다고 댓글을 달았는데, 저렇게 하지 말라고 오지랍을 떤 것이다.

물론 밤 늦게 글이 달렸었고, 그 사이 다행이도 다른 댓글이 달리지 않았고, 출근길에 전철 안에서 세션 단위로 값을 제공하기 때문에 같은 세션에선 거의 발생하지 않는다는...댓글 수정이 아주 불편한 모바일을 감안해서 약간의 변명스러운 댓글을 달았다. 물론 세션 문제를 제외하고, 저런 식의 코딩이 위험하다는 건 변함이 없었지만...

그런데, 모든 댓글에 -1 이 달렸다. 그 커뮤니티에서는 질문/답변 게시판에서 댓글에 1 점 혹은 -1 점을 줄 수 있는데, -1 점이 달린 것이었다. 그 커뮤니티가 나름 프로그래밍 쪽에서는 많은 회원을 가진 곳이긴 하지만 답변 내용이 틀렸다고 저렇게 -1 점을 막 주는 사이트는 아닌데...하면서 의문을 가졌지만 곧 의문이 풀렸다.
다른 사람들끼리 약간의 논쟁(이라 쓰고 싸움이라 읽을 수도 있다)이 벌어지면서 누가 점수를 그렇게 줬는지 알아버렸기 때문이었다. 다만, 그 사람이...내가 다시는 그 사람의 글이나 댓글 등에 반응을 일체 하지 않겠다고 혼자서 결심한 사람이었기에 문제가 되었지만...그래서 이 글을 쓰고 있는 것이기도 하다.

심지어 그 댓글들 때문에 상대에게 별 의미도 없는 -1 점 주는 행동에, 나에게 -1 점을 막 주던 그 사람은 자기에게 -1 점 줬다고 다른 사람들을 비난하기 시작했고, 그 글에서 단 한 번도 나오지 않는 MS SQL Server 에서의 문제까지 가지고 나와서 시덥잖은 변명까지 하기 시작했다.

---

그래서 이 글을 적는다.

먼저, 글의 내용처럼 오라클에 한해서만 예제를 테스트했다. 그리고, 이런 경우는 극히 발생하지 않는다는 것도 덧붙인다. 다만, 난 겪었었던 일이었고...그게 경력이 가지는 무서움이라는 것이다. 일하던 산업군이 다양했던 것도 한 몫 했을 것이다. 특히 금융권 같이...한 번 일하기 시작하면 그 산업군을 벗어나는 경우가 드물고, 코드 자체를 매우 보수적으로 개발하고 운영 역시 그러한 곳에서는 이런 일은 발생하지 않을꺼라 본다. 하지만, 저 질문을 한 사람에게 저걸 쓰는게 뭐 어때서...라고 추천하는 순간 이런 황당한(하지만, 사실 예상할 수도 있다) 경우를 겪을 수 있는, 일종의 버그를 만들 수 있는 코드를 작성하는 개발자 하나를 탄생하게 만드는 것이다. 물론 공개된 곳의 글이기 때문에 한 명에서 그치지 않을 수도 있고(그래서 글 내용만 가지고 기계적인 답변을 달지 말라고 그러는 것이다)...

github 에 전체 코드를 올리면 좋겠지만, 뭐 좋은 코드라고 올리겠는가...그냥 대충만 적는다.

먼저 DB 를 만든다.


~~~sql
create table log (seq int PRIMARY KEY , group_seq int, message varchar2(1000));
create sequence seq_log_seq;
create sequence seq_log_group_seq;
~~~


아래와 같은 두 개의 쿼리를 만든다.

~~~xml
<mapper namespace="com.zepinos.mapper.Log1Mapper">
    <insert id="insertLog1" parameterType="Log">
        insert into log (seq, group_seq, message) values (seq_log_seq.nextval, seq_log_group_seq.nextval, #{message})
    </insert>
</mapper>
~~~
~~~xml
<mapper namespace="com.zepinos.mapper.Log2Mapper">
    <insert id="insertLog2" parameterType="Log">
        insert into log (seq, group_seq, message) values (seq_log_seq.nextval, seq_log_group_seq.currval, #{message})
    </insert>
</mapper>
~~~

그리고 한 개의 서비스를 만들어서 두 쿼리를 실행하도록 한다.

~~~java
@Service
public class TestServiceImpl implements TestService {

    @Autowired
    private Log1Mapper log1Mapper;
    @Autowired
    private Log2Mapper log2Mapper;

    @Transactional
    public void test() {

        Log log = new Log();
        log.setMessage("Test1");

        int result = log1Mapper.insertLog1(log);

        if (result > 0) {

            log.setMessage("Test2");

            log2Mapper.insertLog2(log);

        }

    }

}
~~~

큰 문제가 없다면, 아래와 같이 출력될 것이다.

~~~
1    1    Test1
2    1    Test2
~~~

너무 당연하다. 두번째 컬럼인 group_seq 는 일종의 레코드의 그룹을 알려주기 위해서 하나의 트랜젝션에 같은 값을 가지게 하기 위해서 저렇게 만드는 경우가 있다. 실제 내가 지금 만들고 있는 게임 서버의 로그도 저런 식으로 하나의 트랜젝션(사용자의 한 번의 작업)에 그룹을 구별하기 쉽게 하고, 순서도 별도 컬럼에서 넣기도 한다. 즉, 저런 코딩이 없는 경우가 아니란 것이다. 그리고 원 질문자의 말처럼 currval 은 잘 작동한다.

하지만, 내가 겪은 일은...아쉽게도 저런 식의 코드를 짠 사람이 뭔가 실수해서 발생한 것이 아니었다. 심지어, 문제를 일으킨 코드를 작성한 사람이 잘못한 것도 아니었다. 오히려 괜찮은 실력에, 개발자로써의 자질(게으름!)도 가지고 있었기에 발생했던 것이다.

이걸 쉽게 구현하기 위해서 다음과 같은 코드를 만들었다.

먼저, 그냥 쉽게 하기 위해서 위에서 만든 테이블에 값을 넣는 동일한 쿼리를 만들었다. 물론 내가 겪었던 사이트는 이것조차 만들지 않았다. 빨리 만드느라 이렇게 된 것이니 양해를...

~~~xml
<mapper namespace="com.zepinos.aspect.TimeMapper">
    <insert id="checkTime" parameterType="String">
        insert into log (seq, group_seq, message) values (seq_log_seq.nextval, seq_log_group_seq.nextval, #{message})
    </insert>
</mapper>
~~~

그리고 아래와 같은 클래스를 생성했다. 두둥...

~~~java
@Aspect
public class InsertRunTimeChecker {

    @Autowired
    private TimeMapper timeMapper;

    @Around("execution(* com.zepinos.mapper.*Mapper.insert*(..))")
    public Object insertAround(final ProceedingJoinPoint joinPoint) throws Throwable {

        long time = System.nanoTime();

        Object proceed = joinPoint.proceed();

        timeMapper.checkTime("Spend Time : " + (System.nanoTime() - time));

        return proceed;

    }

}
~~~

뭐...금방 눈치챈 분들도 계시겠지만...

이 코드는 위에서 개발한 두 개의 insert 의 동작 시간을 로그에 추가적으로 남기기 위한 코드다. Aspect 을 이용해서 말이다. 다만, 그 때 당시 이걸 추가했던 사람이 insert 되었던 로그와 같은 그룹으로 넣지 않고 별도로 넣은건 나중에 일괄적으로 삭제를 하려고 했던건지 아니면 혼동을 줄이기 위해서인지는 모르겠다. 어쨌든 두 개의 시퀀스의 용도를 제대로 이해하고 있다면 위와 같은 코드를 만드는게 문제가 될 리 없다.
그리고, 보통은 insert 가 여러번 연달아서 발생하는 경우가 드문 사이트였고...여러 이유에서 저런 코드가 바로 문제시 되지 않았던 것도 서비스 후에 이 문제가 발견된 이유이기도 했다.

당연히 저렇게 해서 값이 입력되면 아래와 같은 결과 형태로 테이블에 저장된다.

~~~
1    1    Test1
2    2    Spend Time : 149358240
3    2    Test2
4    3    Spend Time : 1457061
~~~

자...이렇게 로그가 남게 되면 처음에 의도한 바와 달리 데이터가 이상하게 그룹화 되어서, 저 데이터를 봐야 하는 운영팀이나 사업팀 등은 맨붕에 빠지게 된다. 그나마 실제 상황에서 저런 일이 로그 테이블에서 발생했으니 망정이었지...

이게 왜 짜증나는 경우냐면, 그 때 당시 내가 가이드 코드들을 제공했고, 분명히 insert 시에 &lt;selectKey&gt; 을 이용하는 예제만 존재했기 때문이었다. 물론 나 역시 저런 식의 문제가 생길꺼라곤 생각을 못했지만...흔히 인터넷에 위 질문과 달리 &lt;selectKey&gt; 만 쓰는 이유도, 어떤 선각자(?)가 이런 문제를 예감하고 그랬을런지도 모른다.

왜냐하면, &lt;selectKey&gt; 을 쓸 경우 AOP 등이 끼어들 여지가 없고, 먼저 값을 가져온 뒤에 그 값을 가지고 insert 할 때마다 이용하기 때문에, 고의로 그 값을 변조하지 않는 이상(혹은 정말 실력이 떨어져서 thread 에서 그 값을 공유하는 코딩을 하면 망한다) 이런 문제가 발생하지 않는다. 실제 MyBatis 3 나 iBatis 2 에서는 사용법이 좀 다르긴 하나, MyBatis 3 에서는 <selectKey> 에 여러 값을 조회한 뒤, Bean 에 그 값을 넣어주기 때문에 정말 문제 발생 가능성이 떨어지고, 난 개인적으로 이런 코딩을 방어적인 코딩이라고 부른다.
그런데, 이렇게 코딩하는건...너무 예제도 많고...귀찮아서 여기엔 적지 않겠다.

중요한 건, 분명 Oracle 시퀀스에서 nextval 와 currval 을 insert 에 바로 적을 경우 문제가 발생할 부분이 존재한다는 것이다. 사실 위에서는 극적으로 보이기 위해 AOP 을 썼지만...사실 예전에 문제가 됐을 땐 Service 안에 테이블에 값을 넣는 Mapper 을 호출하도록 중간에 코드를 넣는 형태였다.

그리고, 이런 위험성을 알면서 알려주지 않는 것은...상대가 그걸 받아들일만큼의 실력이 되는지의 여부와는 상관이 없다고 본다. 오히려 문제가 있다고 하는데도 자기 주장 때문에 부득부득 우기는 모습은......그만하련다. 내 짧은 삶에 그런 걸로 시간 낭비하는 것도 짜증난다. 그 사람 입장에서도 평생 이런 문제 안겪을 확률이 높을텐데 내가 왜 나서서 이러고 있는지는 모르겠지만...

어쨌든, Oracle / MyBatis 에서 Sequence 쓸 때 그 값을 재사용한다면 습관적으로 &lt;selectKey&gt; 을 쓰는 걸 권한다. 물론 선실행이 필요한 Sequence 말고도 후실행되는 Auto increment 을 지원하는 다른 DBMS 에서도 그냥 생각없이 &lt;selectKey&gt; 쓰는게 나을 수 있겠다. 굳이 이경우 저경우 다 따질 필요도 없고, 하나의 XML Element 에서 두 번의 쿼리를 통해 이것저것 다 할 수 있어서 코드도 간결해 지니까 말이다.

---

꼬랑지 1. MS SQL Server 는 2012 버전부터 Sequence 을 지원한다. Auto increment 말고 Sequence 쓰는걸 추천한다. 물론 방어적인 이유 때문이다. 본인이 하는 사이트에서 문제 발생하지 않았다면 그냥 계속 써도 무방하다. 나처럼 insert 가 무지막지하게 발생하는 곳에서는 이것 때문에 고생할 수 있겠지만, 평생 이런 경우 안겪는 경우가 더 많을꺼다.

꼬랑지 2. MS SQL Server 에서는 &lt;selectKey&gt; 와 별개와 SELECT SCOPE_IDENTITY() 와 IDENT_CURRENT() 와 관련된 문제가 있다. 그런데, 이걸 위 주제와 연결시키는건 한마디로 지기 싫어서 논점을 흐리는 것에 지나지 않는다고 본다. 이걸 꺼내든 사람은 절대 인정하지 않겠지만...그냥 Sequence 쓰라고 한 마디 하고 싶었지만, 어쨌든 그 커뮤니티에서는 절대 응대하지 않는다...가 나의 개인방침으로 굳어졌기에...물론 여기를 찾아내서 댓글 달아도 응대할 생각 없음. (그 커뮤니티는 내 개인 사이트가 아니기 때문에 그곳이 시끄러워 지는게 절대 좋을게 없다는 판단 때문, 그냥 나만 참으면 되는 것이기에)

꼬랑지 3. 이 글에도 다른 분들의 어거지는 엥간하면 응대 안합니다. 이 문제로 싸워봐야 서로 니 생각이 옳네 내 생각이 옳네...수준이지 건설적이지 못하다고 생각합니다.
