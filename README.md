# Postman_HW_3

## :small_blue_diamond:HW during Vadim Ksendzov course:small_blue_diamond:

```
Environment:
url: http://162.55.220.72:5005
```
### /login
Отправить запрос (POST).
```
{{url}}/login
```
Body form-data:
- login: 123
- password: 456

> Response: { "token": "/s34lfgbj/123/jjd909/78220kjkWpqc1886456383110evny" }

Приходящий токен необходимо передать во все остальные запросы.
```js
const resp = pm.response.json();
pm.environment.set("token", resp.token);
```
************
### /user_info

Отправить запрос (POST)
```
{{url}}/user_info
```
Raw JSON:
```json
{   
  "salary": 40000,
  "name":"Vasya",
  "age": 24,
  "auth_token": "{{token}}"
}
```
> Response: { "person": { "u_age": 24, "u_name": [ "Vasya", 40000, 24 ], "u_salary_1_5_year": 160000 }, "qa_salary_after_12_months": 116000.0, "qa_salary_after_6_months": 80000, "start_qa_salary": 40000 }

1. Статус код 200
```js
pm.test("Status code is 200", function () {
  pm.response.to.have.status(200);
});
```
2. Проверка структуры json в ответе.
```js
const schema = {
  "type": "object",
  "properties": {
    "person": {
      "type": "object",
      "properties": {
        "u_age": {
          "type": "integer"
        },
        "u_name": {
          "type": "array",
          "items": [
            {
              "type": "string"
            },
            {
              "type": "integer"
            },
            {
              "type": "integer"
            }
          ]
        },
        "u_salary_1_5_year": {
          "type": "integer"
        }
      },
      "required": [
        "u_age",
        "u_name",
        "u_salary_1_5_year"
      ]
    },
    "qa_salary_after_12_months": {
      "type": "number"
    },
    "qa_salary_after_6_months": {
      "type": "integer"
    },
    "start_qa_salary": {
      "type": "integer"
    }
  },
  "required": [
    "person",
    "qa_salary_after_12_months",
    "qa_salary_after_6_months",
    "start_qa_salary"
  ]
}

pm.test("Schema is valid", function () {
  pm.response.to.have.jsonSchema(schema)
});
```
3. В ответе указаны коэффициенты умножения salary, напишите тесты по проверке правильности результата перемножения на коэффициент.
```js
const resp = pm.response.json();
const req = JSON.parse(pm.request.body.raw);

const salaryCoeffs = {
  start_qa_salary: 1,
  qa_salary_after_6_months: 2,
  qa_salary_after_12_months: 2.9,
  u_salary_1_5_year: 4
}

Object.entries(salaryCoeffs).forEach(([type, coeff]) => {
  pm.test(`Проверка правильности результата перемножения на коэффициент ${type}`, function () {
    if (type === "u_salary_1_5_year") {
      pm.expect(resp.person[type]).to.eql(req["salary"] * coeff);
    } else {
      pm.expect(resp[type]).to.eql(req["salary"] * coeff);
    }
  });
})
```
4. Достать значение из поля 'u_salary_1.5_year' и передать в поле salary запроса http://162.55.220.72:5005/get_test_user
```js
pm.environment.set("salary", resp.person["u_salary_1_5_year"]);
```
************
### /new_data

Отправить запрос (POST)
```
{{url}}/new_data
```
Body form-data:
- age: 35
- salary: 160000
- name: Oleg
- auth_token: {{token}}

> Response: { "age": 35, "name": "Oleg", "salary": [ 160000, "320000", "480000" ] }

1. Статус код 200
```js
pm.test("Status code is 200", function () {
  pm.response.to.have.status(200);
});
```
2. Проверка структуры json в ответе.
```js
const schema = {
  "type": "object",
  "properties": {
    "age": {
      "type": "integer"
    },
    "name": {
      "type": "string"
    },
    "salary": {
      "type": "array",
      "items": [
        {
          "type": "integer"
        },
        {
          "type": "string"
        },
        {
          "type": "string"
        }
      ]
    }
  },
  "required": [
    "age",
    "name",
    "salary"
  ]
}

pm.test("Schema is valid", function () {
  pm.response.to.have.jsonSchema(schema);
});
```
3. В ответе указаны коэффициенты умножения salary, напишите тесты по проверке правильности результата перемножения на коэффициент.
```js
resp.salary.forEach((item, index) => {
  pm.test(`Проверка правильности результата перемножения на коэффициент ${index + 1}`, function () {
      pm.expect(req.salary * (index + 1)).to.eql(+item);
  });
})
```
4. проверить, что 2-й элемент массива salary больше 1-го и 0-го.
```js
pm.test("Проверить, что 2-й элемент массива salary больше 0-го и 1-го", function () {
  pm.expect(+resp.salary[0]).to.be.below(+resp.salary[2]);
  pm.expect(+resp.salary[1]).to.be.below(+resp.salary[2]);
});
```
************
### /test_pet_info

Отправить запрос (POST)
```
{{url}}/test_pet_info
```
Body form-data:
- age: 35
- weight: 60
- name: Olga
- auth_token: {{token}}

> Response: { "age": 25, "daily_food": 0.72, "daily_sleep": 150.0, "name": "Olga" }

1. Статус код 200
```js
pm.test("Status code is 200", function () {
  pm.response.to.have.status(200);
});
```
2. Проверка структуры json в ответе.
```js
const schema = {
  "type": "object",
  "properties": {
    "age": {
      "type": "integer"
    },
    "daily_food": {
      "type": "number"
    },
    "daily_sleep": {
      "type": "number"
    },
    "name": {
      "type": "string"
    }
  },
  "required": [
    "age",
    "daily_food",
    "daily_sleep",
    "name"
  ]
}

pm.test("Schema is valid", function () {
  pm.response.to.have.jsonSchema(schema);
});
```
3. В ответе указаны коэффициенты умножения weight, напишите тесты по проверке правильности результата перемножения на коэффициент.
```js
const weigthCoeffs = {
  daily_food: 0.012,
  daily_sleep: 2.5,
}

Object.entries(weigthCoeffs).forEach(([type, coeff]) => {
  pm.test(`Проверка правильности результата перемножения на коэффициент ${type}`, function () {
    pm.expect(resp[type]).to.eql(req.weight * coeff);
  });
});
```
************
### /get_test_user

Отправить запрос (POST)
```
{{url}}/get_test_user
```
Body form-data:
- age: 25
- salary: {{salary}}
- name: Zina
- auth_token: {{token}}

> Response: { "age": "25", "family": { "children": [ [ "Alex", 24 ], [ "Kate", 12 ] ], "u_salary_1_5_year": 640000 }, "name": "Zina", "salary": 160000 }

1. Статус код 200
```js
pm.test("Status code is 200", function () {
  pm.response.to.have.status(200);
});
```
2. Проверка структуры json в ответе.
```js
const schema = {
  "type": "object",
  "properties": {
    "age": {
      "type": "string"
    },
    "family": {
      "type": "object",
      "properties": {
        "children": {
          "type": "array",
          "items": [
            {
              "type": "array",
              "items": [
                {
                  "type": "string"
                },
                {
                  "type": "integer"
                }
              ]
            },
            {
              "type": "array",
              "items": [
                {
                  "type": "string"
                },
                {
                  "type": "integer"
                }
              ]
            }
          ]
        },
        "u_salary_1_5_year": {
          "type": "integer"
        }
      },
      "required": [
        "children",
        "u_salary_1_5_year"
      ]
    },
    "name": {
      "type": "string"
    },
    "salary": {
      "type": "integer"
    }
  },
  "required": [
    "age",
    "family",
    "name",
    "salary"
  ]
}

pm.test("Schema is valid", function () {
  pm.response.to.have.jsonSchema(schema);
});
```
3. Проверить что значение поля name = значению переменной name из окружения.
```js
pm.environment.set("name", resp.name);

pm.test("Проверить что значение поля name = значению переменной name из окружения", function () {
  pm.expect(resp.name).to.eql(pm.environment.get("name"));
});
```
4. Проверить что значение поля age в ответе соответсвует отправленному в запросе значению поля age.
```js
pm.test("Значение поля age в ответе соответсвует отправленному в запросе значению поля age", function () {
  pm.expect(resp.age).to.eql(req.age);
});
```
