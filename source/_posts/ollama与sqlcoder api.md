---
title: ollama与sqlcoder api
date: 2024-07-28 09:57:00 
category: 人工智能
tags: [fastapi,sqlcoder]
---

# ollama

## 安装

```
略
ollama pull sqlcoder-7b
```



## 编写API

**安装环境**

```
pip install fastapi uvicorn ollama
```



```py
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import ollama

app = FastAPI()

class SQLRequest(BaseModel):
    question: str
    schema: str

@app.post("/generate_sql")
async def generate_sql(request: SQLRequest):
    prompt = f"""
    ### Instructions:
    Your task is to convert a question into a SQL query, given a Postgres database schema.
    Adhere to these rules:
    - **Deliberately go through the question and database schema word by word** to appropriately answer the question
    - **Use Table Aliases** to prevent ambiguity. For example, `SELECT table1.col1, table2.col1 FROM table1 JOIN table2 ON table1.id = table2.id`.
    - When creating a ratio, always cast the numerator as float

    ### Input:
    Generate a SQL query that answers the question `{request.question}`.
    This query will run on a database whose schema is represented in this string:
    `{request.schema}`

    ### Response:
    Based on your instructions, here is the SQL query I have generated to answer the question `{request.question}`:
    ```sql
    """

    try:
        response = ollama.chat(model='mannix/defog-llama3-sqlcoder-8b:latest', messages=[{'role': 'user', 'content': prompt}])
        return {"sql_query": response['message']['content']}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
    
```

**启动**

python main.py

## 测试api

使用postmain

![img](https://clyhs.github.io/images/ollama/sqlcoder01.png)




