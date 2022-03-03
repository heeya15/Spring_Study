## 연관관계

- 객체는 참조를 사용해서 다른 객체와 연관관계를 가지고 참조에 접근해서 **연관된 객체를 조회**한다. 반면에 테이블은 외래 키를 사용해서 다른 테이블과 연관관계를 가지고 **조인을 해서 연관된 테이블을 조회**한다.



## 객체를 테이블에 맞추어 모델링

```
class Member {
    String id; // MEMBER_ID 칼럼사용
    Long teamId;  // TEAM_ID PK 칼럼사용
    String username; //USERNAME 칼럼사용
}

class Team {
    Loing id; //TEAM_ID PK 사용
    String name; //NAME 칼럼사용
}
```

- **관계형 데이터베이스는 조인이라는 기능이 있음으로 외래 키의 값을 그대로 보관해도 된다. 하지만 객체는 "연관된 객체의 참조를 보관"해야한다 다음 처럼 구조를 통해 연관돤 객체를 찾을 수 있다.**



## 객체지향 모델링

```
class Member {
    String id; // MEMBER_ID 칼럼사용
    Team team; // 참조로 연관관계를 맺는다.
    String username; //USERNAME 칼럼사용
}

class Team {
    Loing id; //TEAM_ID PK 사용
    String name; //NAME 칼럼사용
}
```

- **외래 키의 값을 그대로 보관하는 것이 아니라 "연관된 Team의 참조를 보관"한다.**
- 그런데 이처럼 객체지향 모델링을 사용하면 객체를 테이블에 저장하거나 조회하기가 쉽지 않다.
  - Member 객체는 team 필드로 연관관계를 맺고 Member 테이블은 TEAM_ID 외래 키로 연관관계를 맺기 때문이다.
  - 반면에 테이블은 참조가 필요 없고 외래 키만 있으면 된다. 결국 개발자가 중간에 변화 역할을 해야한다.



### 저장

- **객체를 데이터베이스에 저장**하면 **team 필드를 TEAM_ID 외래 키 값으로 변환** 해야한다.
- 다음처럼 외래 키 값을 찾아서 INSERT SQL을 만들어야 한다.

```
member.getId(); //Member_ID PK에 저장
member.getTeam.getId(); //Team_ID PK에 저장
member.getUsername(); //Username 칼럼에 저장
```



### 조회

- 조회할 때는 **TEAM_ID 외래 키 값**을 Member 객체의 **team 참조로 변환**해서 객체에 보관해야 한다.

```
SLELECT M.*, T.*
    FROM MEMBER M
    JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID

public Member find(String memberId) {
    //SQL 실행
    Member member = new Member();
    ...
    
    //데이터베이스에 조회한 회원 관련 정보를 모두 입력
    Team team = new Team();

    member.setTeam(team);
    return member;
}
```

- **이러한 과정들은 모두 패러다임 불일치를 해결햐려고 소모하는 비용이다. 만약 자바 컬렉션에 회원 객체를 저장한다면 이런 비용이 전혀들지 않는다.**



## JPA와 연관관계

- JPA는 연관관계와 관련된 패러다임의 불일치 문제를 해결해준다.

```
member.setTeam(team); //회원과 팀 연관관계 설정
jpa.persust(member); //회원과 연관관계 함께 저장
```

- 개발자는 **회원과 팀의 관계를 설정**하고 **회원 객체를 저장**하면 된다.
- JPA는 team의 참조를 **외래 키로 변환**해서 적적한 INSERT SQL을 데이터베이스에 전달한다.



## 객체 그래프 탐색

- 객체에서 회원이 소속된 팀을 조회할 때는 다음처럼 참조를 사용해서 연관된 팀을 찾으면 되는데 이것을 객체 그래프 탐색이라고 한다.

```
Team team = member.getTeam(); // 객체를 참조해서 객체 그래프 탐색
member.getOrder().goetOderItem().... // 자율운 객체 그래프 탐색

member.getOder(); // null 인경우에는 객체 그래프 탐색을 할 수 가 없다.
```

- 에를들어 MemberDAO에서 member 객체를 조회할 때 이런 SQL을 실행해서 회원과 팀에 대한 데이터만 조회했다면 member.getTeam()은 성공하지만 다음처럼 다른 **객체 그래프는 데이터가 없다면 탐색할 수 없다**.
- **SQL을 직접 다루면 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색할 수 있을지 정해야한다.**
- **이것은 객체지향 개발자에겐 너무 큰 제약이다. 객체 그래프가 다른데 언제 끊어질지 모를 객체 그래프를 함부로 탐색할 수는 없기 때문이다.**