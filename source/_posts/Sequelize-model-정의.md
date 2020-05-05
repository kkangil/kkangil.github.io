---
title: Sequelize model 정의
date: 2019-04-28 14:45:40
categories:
  - Sequelize
tags:
  - Sequelize
  - MySQL
  - model 정의
  
toc: true
widgets:
  - type: toc
    position: right
  - type: categories
    position: right
  - type: tags
    position: right
  - type: adsense
    position: right
    client_id: ca-pub-5445993070474035
    slot_id: ''
---
## sequelize model 정의
- sesquelize model 을 정의하는 방법은 여러가지 있지만, sequelize 의 `define` 메소드를 이용해 정의하고자 한다.
- model이란 데이터베이스의 테이블에 해당되며, 객체내의 key 값은 컬럼으로 생성된다.
```javascript
return sequelize.define(
  "User",
  {
    username: {
      type: DataTypes.STRING(128)
    },
    job: {
      type: DataTypes.STRING
    }
  }
)
```

<!-- more -->

- `define` 함수의 첫번째 파라미터는 테이블의 이름에 해당된다. 또한 express 프로젝트에서도 해당 이름으로 데이터베이스에 접근할 수 있다.
- 두번째 파라미터는 컬럼을 정의하는 것이다. 위의 예시는 가장 기본적인 컬럼을 생성한 것이고 나머지 옵션들은 밑에서 확인해 보도록 한다.


## column option
- sequelize model 생성시 sequelize 는 고유 키값을 정의해 주지 않아도, 다른 설정이 없다면 id 로 생성해주며 row 생성시 자동으로 1씩 증가한다.
- id 이외에 createdAt(생성일), updatedAt(수정일) 도 같이 생성해준다.
- defaultValue: row가 생성될때 기본값을 설정해줄 수 있다. (etc. Sequelize.NOW)
- allowNull: false로 설정해주면 빈값으로 생성시 에러가 난다.(default true)
- unique: 테이블내의 고유한 값(boolean)
- primaryKey: 고유 키값 설정 여부
- autoIncrement: 자동으로 값을 증가시켜준다. (Integer 에서만 사용 가능)
- field: 객체 키값과 다르게 custom으로 컬럼명을 사용할 수 있게해준다.
- comment: 해당 컬럼에 대한 설명을 달 수 있다. 컬럼 생성에 영향을 미치지는 않는다. 주석같은 개념


## Getters & setters
- 컬럼 option에 getter와 setter를 추가해 줄 수 있다.
```javascript
return sequelize.define(
  "User",
  {
    username: {
      type: DataTypes.STRING(128),
      allowNull: false,
      get() {
        const job = this.getDataValue('job')
        return `${this.getDataValue('username')} (${this.getDataValue('job')})`
      }
    },
    job: {
      type: DataTypes.STRING,
      allowNull: false,
      set(job) {
        this.setDataValue('job', job.toUpperCase())
      }
    }
  }
)

const userSample = async () => {
  const user = await User.create({ username: 'kkangil', job: 'developer' })
  console.log(user.get('username')) // kkangil (DEVELOPER)
  console.log(user.get('job')) // DEVELOPER
}
```
- `getDataValue` 를 사용하여 자신의 컬럼 뿐만 아니라 테이블의 컬럼 데이터도 가져올 수 있다.
- `setDataValue` 를 사용하여 생성이나 수정 시 데이터 값을 수정, 변경할 수 있다. 
- 컬럼 객체 내부에 설정해 주지 않고, `define` 함수의 세번째 파라미터로도 사용이 가능하다.
```javascript
return sequelize.define(
  "User",
  {
    username: {
      type: DataTypes.STRING(128),
      allowNull: false
    },
    job: {
      type: DataTypes.STRING,
      allowNull: false
    }
  }, {
    getterMethods: {
      getUser() {
        return `${this.username} (${this.job})`
      }
    }
  }
)
```

## validate column
- 데이터 타입 이외에도 `validate` 를 사용해서 유효성 확인 후 에러를 반환해 줄 수 있다.
```javascript
return sequelize.define(
  "Foo",
  {
    bar: {
      type: DataTypes.STRING,
      validate: {
        is: /^[a-z]+$/i, // 정규식 사용해서 유효성 확인
        not: /^[a-z]+$/i,
        isEmail: true, // 이메일 유효성 확인
        isInt: true,
        notNull: true,
        isNull: true,
        notEmpty: true, // string 빈값 확인
        equals: 'specific value', // 특정 값으로만 생성 가능
        contains: 'foo', // 해당 값을 포함하고 있는지 확인
        notContains: 'bar',
        notIn: [['foo', 'bar']],
        isIn: [['foo', 'bar']],
        len: [2,10], // 2자리 ~ 10자리
        max: 23,
        min: 10,
        isCreditCard: true // 신용카드 유효성 확인
      }
    }
  }
)
```

- 제공되는 `validate` 가 아닌 직접 만들어서 사용하는 기능도 제공한다.
```javascript
bar: {
  type: DataTypes.INTEGER,
  validate: {
    isEven(value) {
      if (parseInt(value) % 2 !== 0) {
        throw new Error('Only even values are allowed!');
      }
    }
  }
}
```
- throw new Error의 메시지가 에러로 리턴된다.
- 직접 만들어서 사용하지 않아도 메시지를 설정해 줄 수 있다.
```javascript
bar: {
  type: DataTypes.INTEGER,
  validate: {
    notNull: {
      msg: "Must"
    },
    isIn: {
      args: [['en', 'zh']],
      msg: "Must be English or Chinese"
    }
  }
}
```
- `allowNull` 을 사용한다면 `notNull` 의 `msg`를 설정해주면 에러메시지로 사용가능하다.

```javascript
return sequelize.define(
  "User",
  {
    username: {
      type: DataTypes.STRING(128),
      allowNull: false
    },
    job: {
      type: DataTypes.STRING,
      allowNull: false
    }
  }, {
    validate: {
      checkUsernameAndJob() {
        if (!(this.username && this.job)) {
          throw new Error('이름과 직업을 입력해주세요.')
        }
      }
    }
  }
)
```

- `define` 함수의 세번째 파라미터 객체에 validate 를 설정해주면 하나의 컬럼이 아닌 테이블의 모든 컬럼의 유효성은 같이 확인 할 수 있다.


## model configuration
- `define` 함수의 세번째 파라미터 객체의 설정값
```javascript
return sequelize.define(
  ..., {
    modelName: 'bar', // 모델 이름 설정
    timestamps: false, // createdAt, updatedAt 생성하지 않음
    paranoid: true, // 데이터를 삭제하지 않고 현재 시간으로 deletedAt 데이터가 추가된다.
    underscored: true, // 자동으로 컬럼명을 snake 네임으로 변경한다.
    tableName: 'my_bar', // 테이블 이름 설정
    createdAt: false, // createdAt 사용하지 않음
    updatedAt: 'updateTimestamp', // updatedAt 컬럼명 정의
    deletedAt: 'destroyTime', // deletedAt 컬럼명 정의 (paranoid가 true로 설정되어 있어야함.)
  }
)
```

참고 [sequelize docs](http://docs.sequelizejs.com/manual/models-definition.html)
