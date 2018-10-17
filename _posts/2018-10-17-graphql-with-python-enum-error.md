---
title: "Python: GraphQL + SQLAlchemy > Model(Enum) Error 대처"
excerpt_separator: <!--more-->
categories:
  - Python
  - GraphQL
  - SQLAlchemy
  - GraphQL Enum
tags: 
  - python
  - graphql
  - sqlalchemy
  - flask
  - enum
  - error
---

### GraphQL + SQLAlchemy 리플렉션시, Enum이 있는 모델 에러 발생시 대처법
그래프QL에서 `schema.py`를 작성시, 처음부터 선언하는 방식이 아닌 `SQLAlchemy` 모델 리플렉션을 사용하여 스키마를 빌드하면 매우 편리하다. 하지만, 모델에 Enum 컬럼이 존재하면, 해당 모델은 문자열로 처리되어, `type.name`이 `None`을 반환, 열거형 변환이 실패한다. 에러 예시는 다음과 같다.

- Error
```bash
enum_class = super().__new__(metacls, cls, bases, classdict)
TypeError: type.__new__() argument 1 must be str, not None
```
해당 에러를 뿜으면서 `app.py`가 실행안된다.

<!--more-->
- schema.py
<br/>
```python
import graphene
from graphene import relay
from graphene_sqlalchemy import SQLAlchemyConnectionField, SQLAlchemyObjectType, utils, converter
# Model내부에 Enum 컬럼이 있음.
from model import ModelWithEnum as ModelWithEnumModel
# GraphQL용 모델선언
class ModelWithEnum(SQLAlchemyObjectType):
    class Meta:
        model = ModelWithEnumModel
        interfaces = (relay.Node, )
```

---
### Solution
해당 부분을 해결하기 위해선, `graphene_sqlalchemy`를 수정해줘야한다. 해당 패키지를 직접 수정해줘도 되는데, 나 같은 경우 소스를 `git`으로 관리하기때문에, 프로젝트 디렉토리에 <kbd>Copy</kbd> & <kbd>Paste</kbd>해서 따로 관리하기로 했다.

#### 주요 패키지 버전들
```bash
Flask               1.0.2
Flask-GraphQL       2.0.0
graphene            2.1.3
graphene-sqlalchemy 2.1.0 (프로젝트에 복사 후 언인스톨)
graphql-core        2.1
graphql-relay       0.4.5
graphql-server-core 1.1.1
mysqlclient         1.3.13
PyMySQL             0.9.2
sqlacodegen         2.0.1
SQLAlchemy          1.1.14
```

#### 해결순서
1) 프로젝트로 패키지 복사 (후미에 `_customized`라고 붙여주었다) 
```bash
cp -rf ~/.virtualenvs/[가상환경이름]/lib/python3.6/site-packages/graphene_sqlalchemy path/to/project/graphene_sqlalchemy_customized
pip uninstall graphene-sqlalchemy -y
```

2) 해당 패키지의 `converter.py` 수정
- Before
```python
# converter.py
@convert_sqlalchemy_type.register(types.Enum)
def convert_enum_to_enum(type, column, registry=None):
    try:
        items = type.enum_class.__members__.items()
    except AttributeError:
        items = zip(type.enums, type.enums)
    return Field(
        Enum(type.name, items),
        description=get_column_doc(column),
        required=not (is_column_nullable(column)),
    ) 
```

- After
```python
# converter.py
@convert_sqlalchemy_type.register(types.Enum)
def convert_enum_to_enum(type, column, registry=None):
    try:
        items = type.enum_class.__members__.items()
    except AttributeError:
        items = zip(type.enums, type.enums)
    # 변경단
    enum_name = type.name or "{}_{}".format(column.table, column.name)
    return Field(
        Enum(enum_name, items),
        description=get_column_doc(column),
        required=not (is_column_nullable(column)),
    )
```
해당 함수의 `type.name`을 이와같이 수정하면 정상화된다.






