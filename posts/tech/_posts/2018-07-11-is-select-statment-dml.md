---
title: select statement는 DML인가?
---

현재까지의 결론은 '확실치않다'이다.
이 문제에 대해서 사람들의 의견은 다양한 것 같다.
위키피디아는 SELECT 문서[^1]에서 SELECT가 DQL로 부를 수 있다고 말하고 있다. 그렇다고 DML이 아니라는 말을 하는거 같지는 않다.
위키피디아 DML 문서[^2]는 그래도 SELECT가 DML이라고 한다.

> Read-only selecting of data is sometimes distinguished as being part of a separate data query language (DQL), but it is closely related and sometimes also considered a component of a DML; some operators may perform both selecting (reading) and writing.

구글이나 스텍오버플로우에도 `is select dml?` 이런 식으로 검색해보면 신통한 답이 존재하진 않는 것 같다. 그래서 표준 문서를 찾아서 이 문제를 해결해보고자 하였다.

표준 문서를 본 결과, 이 혼란이 일어난 가장 큰 이유는 SQL표준에서 DML(data manipulation language)라는 표현을 사용하지 않기 때문이라고 생각되었다.
ISO/IEC 9075는 Information technology — Database languages — SQL 을 위한 표준인데, 여기에서 DML을 찾아보았지만 찾을 수 없었다. 대신 가장 유사한 표현은 `SQL-data statements`였다. ISO/IEC 9075-1에 따르면 이는 다음과 같다.

> SQL-data statements that perform queries, and insert, update, and delete operations on tables. Execution of an SQL-data statement is capable of affecting more than one row, of more than one table.

ISO/IEC 9075-2에서는 목차 구성을 통해 조금 엿볼 수 있었다. `Data manipulation` 항목 안에 `<select statment: single row>` 가 포함되어 있고, `delete`, `insert`, `update` 도 여기에 포진하고 있다. 따라서 SQL-data statment 중 Data manipulation에 select가 포함된다는 말은 맞는 것으로 보인다.
하지만 data manipulation에 select가 있다는 것이 select가 DML이라는 것을 확실하게 해주는 것은 아니다. 일단 DML이라는 용어 자체가 아예 없다. 이 둘의 관계는 무엇인가? 공신력은 없지만 한 페이지는 다음과 같이 소개하고 있다.

> SQL-Data Statements perform query and modification on database tables and columns. This subset of SQL is also called the Data Manipulation Language for SQL (SQL DML).[^3]

일단 잠정적으로 SELECT를 DML에 들어가는 것으로 보겠다. 잘 아시는 분 있으면 가르침을...

[^1]: https://en.wikipedia.org/wiki/Select_(SQL)
[^2]: https://en.wikipedia.org/wiki/Data_manipulation_language
[^3]: http://www.firstsql.com/tutor1.htm
