---
title: 파이썬과 FastAPI 공부하기 (1) — 환경 세팅과 첫 API
date: 2026-10-14 12:00:00 +0900
categories: [Study, Python]
tags: [python, fastapi, uv, backend]
---

## 1. FastAPI란

FastAPI는 파이썬으로 API 서버를 만드는 웹 프레임워크입니다. Starlette(웹 처리)과 Pydantic(데이터 검증) 위에 만들어졌습니다.

FastAPI는 파이썬의 타입 힌트를 그대로 활용합니다. 함수 인자에 타입을 적으면 FastAPI가 요청 값을 그 타입으로 변환하고 검증합니다. 같은 정보로 API 문서도 자동으로 만들어 줍니다.

## 2. uv로 프로젝트 만들기

이 시리즈에서는 책을 등록하고 조회하는 독서 기록 API를 만들면서 FastAPI의 기능을 하나씩 살펴봅니다.

uv는 파이썬 패키지 매니저입니다. 패키지 설치(`pip`), 가상환경(`venv`), 파이썬 버전 관리(`pyenv`)를 하나의 도구로 처리합니다.

```bash
uv init reading-log --app --no-package --python 3.13
cd reading-log
uv add "fastapi[standard]"
```

`--python 3.13`을 지정하면 이 프로젝트에서만 파이썬 3.13을 사용합니다. 시스템에 설치된 버전과는 별개로 관리됩니다. `fastapi[standard]`는 FastAPI와 함께 개발 서버(uvicorn), CLI 등 자주 쓰는 의존성을 같이 설치합니다.

프로젝트를 만들면 아래 파일이 생깁니다.

| 파일 | 역할 | npm으로 치면 |
|---|---|---|
| `pyproject.toml` | 프로젝트 정보와 의존성 목록 | `package.json` |
| `uv.lock` | 설치된 버전 고정 | `package-lock.json` |
| `.venv/` | 가상환경 (패키지가 설치되는 곳) | `node_modules/` |

## 3. 첫 번째 API 만들기

`main.py`를 아래처럼 작성합니다.

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"message": "Hello, Reading Log!"}
```

`FastAPI()`로 앱 객체를 만들고, `@app.get("/")` 데코레이터로 `GET /` 요청을 아래 함수에 연결합니다. 함수가 딕셔너리를 반환하면 FastAPI가 JSON으로 변환해 응답합니다.

서버는 아래 명령어로 실행합니다.

```bash
uv run fastapi dev main.py
```

`fastapi dev`는 코드를 저장할 때마다 서버를 다시 시작하는 개발 모드입니다. 실행 후 `http://127.0.0.1:8000`에 접속하면 `{"message": "Hello, Reading Log!"}`가 나타납니다. 운영 환경에서는 `fastapi run`을 사용합니다.

## 4. 경로 파라미터와 쿼리 파라미터

FastAPI는 함수 인자를 보고 값을 어디서 가져올지 정합니다.

```python
books = [
    {"id": 1, "title": "데미안", "author": "헤르만 헤세"},
    {"id": 2, "title": "채식주의자", "author": "한강"},
]


@app.get("/books")
def list_books(author: str | None = None):
    if author is None:
        return books
    return [book for book in books if book["author"] == author]


@app.get("/books/{book_id}")
def get_book(book_id: int):
    for book in books:
        if book["id"] == book_id:
            return book
    raise HTTPException(status_code=404, detail="Book not found")
```

경로에 `{book_id}`로 적은 인자는 **경로 파라미터**가 됩니다. 경로에 없는 `author`는 **쿼리 파라미터**가 되어 `/books?author=한강`처럼 받습니다. 기본값을 `None`으로 두면 생략할 수 있는 선택 파라미터가 됩니다.

`book_id: int`로 선언했기 때문에 `/books/1`의 `"1"`은 정수 `1`로 변환됩니다. `/books/abc`처럼 정수로 바꿀 수 없는 값이 들어오면 FastAPI가 422 에러를 반환합니다. 검증 코드를 따로 작성하지 않아도 됩니다.

찾는 데이터가 없을 때는 `HTTPException`을 발생시킵니다. `from fastapi import FastAPI, HTTPException`으로 가져와서 사용합니다.

> `str | None` 문법은 파이썬 3.10부터 사용할 수 있습니다. 이전 버전에서는 `Optional[str]`을 씁니다.
{: .prompt-info }

## 5. POST 요청 받기

데이터를 추가하는 API는 `@app.post`로 만듭니다.

```python
@app.post("/books", status_code=201)
def add_book(book: dict):
    new_book = {"id": len(books) + 1, **book}
    books.append(new_book)
    return new_book
```

경로에도 없고 기본 타입(`int`, `str` 등)도 아닌 인자는 요청 본문으로 받습니다. `status_code=201`로 성공 응답 코드를 지정할 수 있습니다. `**book`은 딕셔너리를 펼치는 문법으로, JS의 `{ ...book }`과 같습니다.

지금처럼 `dict`로 받으면 어떤 값이든 그대로 저장됩니다. 필수 필드나 타입을 검사하려면 Pydantic 모델을 사용합니다.

## 6. 자동 생성 문서

서버를 실행한 상태에서 아래 주소에 접속하면 API 문서를 볼 수 있습니다.

| 주소 | 내용 |
|---|---|
| `/docs` | Swagger UI. 문서에서 바로 요청을 보내 볼 수 있음 |
| `/redoc` | ReDoc. 읽기용 문서 |
| `/openapi.json` | 원본 OpenAPI 명세 |

FastAPI가 데코레이터와 타입 힌트를 읽어 OpenAPI 명세를 만들기 때문에, API를 추가하거나 타입을 바꾸면 문서에도 바로 반영됩니다. 명세는 파일로 저장되지 않고 서버가 실행되는 동안 메모리에서 생성됩니다.
