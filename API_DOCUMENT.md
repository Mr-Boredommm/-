# MathPop API接口文档

## 文档说明
本文档详细描述了MathPop小学生速算练习软件的API接口规范，供前后端开发人员参考。

## 接口通用规范

### 基础URL
- **开发环境**: `http://localhost:3000/api`
- **测试环境**: `https://test-api.mathpop.edu/api`
- **生产环境**: `https://api.mathpop.edu/api`

### 请求方法
- GET: 获取资源
- POST: 创建资源
- PUT: 更新资源
- DELETE: 删除资源

### 响应格式
所有API返回统一的JSON格式：

```json
{
  "code": 200,           // 状态码，200表示成功，非200表示失败
  "message": "success",  // 状态描述
  "data": { }            // 返回的数据，失败时可能为null或包含错误详情
}
```

### 状态码说明
- 200: 请求成功
- 400: 请求参数错误
- 401: 未授权（未登录或token失效）
- 403: 权限不足
- 404: 资源不存在
- 500: 服务器内部错误

### 认证方式
采用JWT (JSON Web Token) 认证机制：
- 登录后获取token
- 后续请求在Header中添加：`Authorization: Bearer {token}`

## 用户系统接口

### 1. 用户注册
- **接口**: POST /user/register
- **描述**: 新用户注册
- **请求参数**:
```json
{
  "username": "string",    // 用户名，6-20个字符
  "password": "string",    // 密码，8-20个字符，包含数字和字母
  "email": "string",       // 邮箱
  "userType": "enum",      // 用户类型：student(学生)/parent(家长)/teacher(教师)
  "grade": "number",       // 年级，1-6表示小学一年级到六年级（学生必填）
  "parentCode": "string"   // 家长绑定码（可选，用于绑定已有家长）
}
```
- **响应**:
```json
{
  "code": 200,
  "message": "注册成功",
  "data": {
    "userId": "string",
    "username": "string",
    "userType": "enum",
    "token": "string"      // JWT令牌
  }
}
```

### 2. 用户登录
- **接口**: POST /user/login
- **描述**: 用户登录接口
- **请求参数**:
```json
{
  "username": "string",    // 用户名或邮箱
  "password": "string"     // 密码
}
```
- **响应**:
```json
{
  "code": 200,
  "message": "登录成功",
  "data": {
    "userId": "string",
    "username": "string",
    "userType": "enum",
    "token": "string",     // JWT令牌
    "expireTime": "number" // 过期时间戳（毫秒）
  }
}
```

### 3. 获取用户信息
- **接口**: GET /user/profile
- **描述**: 获取当前登录用户信息
- **请求头**: Authorization: Bearer {token}
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "userId": "string",
    "username": "string",
    "email": "string",
    "userType": "enum",
    "grade": "number",
    "createTime": "string",
    "lastLoginTime": "string",
    "studyStats": {
      "totalExerciseCount": "number", // 总练习题数
      "correctRate": "number",        // 正确率
      "totalStudyTime": "number",     // 总学习时长（分钟）
      "consecutiveDays": "number"     // 连续学习天数
    }
  }
}
```

### 4. 更新用户信息
- **接口**: PUT /user/profile
- **描述**: 更新当前用户基本信息
- **请求头**: Authorization: Bearer {token}
- **请求参数**:
```json
{
  "email": "string",    // 可选
  "grade": "number",    // 可选
  "avatar": "string"    // 头像URL，可选
}
```
- **响应**:
```json
{
  "code": 200,
  "message": "更新成功",
  "data": null
}
```

### 5. 修改密码
- **接口**: PUT /user/password
- **描述**: 修改当前用户密码
- **请求头**: Authorization: Bearer {token}
- **请求参数**:
```json
{
  "oldPassword": "string",
  "newPassword": "string"
}
```
- **响应**:
```json
{
  "code": 200,
  "message": "密码修改成功",
  "data": null
}
```

### 6. 家长绑定学生
- **接口**: POST /user/bind-student
- **描述**: 家长绑定学生账号
- **请求头**: Authorization: Bearer {token}
- **请求参数**:
```json
{
  "studentCode": "string"  // 学生的绑定码
}
```
- **响应**:
```json
{
  "code": 200,
  "message": "绑定成功",
  "data": {
    "studentId": "string",
    "studentName": "string"
  }
}
```

## 速算练习模块接口

### 1. 获取练习类型
- **接口**: GET /exercise/types
- **描述**: 获取所有可用的练习类型和模式
- **请求头**: Authorization: Bearer {token}
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "typeId": "string",
      "typeName": "string",  // 如：基础练习、综合练习、专项训练等
      "description": "string",
      "modes": [
        {
          "modeId": "string",
          "modeName": "string", // 如：闯关模式、计时模式等
          "description": "string"
        }
      ],
      "difficultyLevels": [
        {
          "levelId": "string",
          "levelName": "string", // 如：简单、中等、困难
          "targetGrade": "number" // 适合年级
        }
      ]
    }
  ]
}
```

### 2. 生成练习题目
- **接口**: POST /exercise/generate
- **描述**: 根据条件生成练习题目
- **请求头**: Authorization: Bearer {token}
- **请求参数**:
```json
{
  "typeId": "string",       // 练习类型ID
  "modeId": "string",       // 练习模式ID
  "levelId": "string",      // 难度级别ID
  "count": "number",        // 生成题目数量，默认10道
  "specificOperation": "string"  // 可选，针对特定运算类型（如加法、乘法等）
}
```
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "exerciseId": "string",  // 练习ID，用于后续提交答案
    "title": "string",       // 练习标题
    "description": "string", // 练习说明
    "timeLimit": "number",   // 时间限制（秒），计时模式有效
    "problems": [
      {
        "problemId": "string",
        "content": "string",  // 题目内容，如"1+2="
        "options": ["string"], // 选择题时的选项，非选择题为null
        "problemType": "enum"  // 题目类型：calculation(计算题)/choice(选择题)
      }
    ]
  }
}
```

### 3. 提交练习答案
- **接口**: POST /exercise/submit
- **描述**: 提交一次练习的所有答案
- **请求头**: Authorization: Bearer {token}
- **请求参数**:
```json
{
  "exerciseId": "string",
  "usedTime": "number",      // 用时（秒）
  "answers": [
    {
      "problemId": "string",
      "answer": "string",    // 用户答案
      "usedTime": "number"   // 单题用时（毫秒）
    }
  ]
}
```
- **响应**:
```json
{
  "code": 200,
  "message": "提交成功",
  "data": {
    "exerciseId": "string",
    "score": "number",        // 得分
    "correctCount": "number", // 正确题数
    "totalCount": "number",   // 总题数
    "correctRate": "number",  // 正确率
    "usedTime": "number",     // 总用时（秒）
    "evaluation": "string",   // 评价
    "wrongProblems": [        // 错题列表
      {
        "problemId": "string",
        "content": "string",
        "userAnswer": "string",
        "correctAnswer": "string"
      }
    ],
    "nextLevelUnlocked": "boolean" // 闯关模式是否解锁下一关
  }
}
```

### 4. 获取练习记录
- **接口**: GET /exercise/records
- **描述**: 获取用户的练习记录
- **请求头**: Authorization: Bearer {token}
- **请求参数**:
```
page: number       // 页码，默认1
pageSize: number   // 每页条数，默认10
typeId: string     // 可选，筛选特定练习类型
startDate: string  // 可选，开始日期，格式YYYY-MM-DD
endDate: string    // 可选，结束日期，格式YYYY-MM-DD
```
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "total": "number",
    "page": "number",
    "pageSize": "number",
    "records": [
      {
        "recordId": "string",
        "exerciseId": "string",
        "typeName": "string",
        "modeName": "string",
        "levelName": "string",
        "score": "number",
        "correctRate": "number",
        "usedTime": "number",
        "createTime": "string"
      }
    ]
  }
}
```

### 5. 获取练习详情
- **接口**: GET /exercise/records/{recordId}
- **描述**: 获取特定练习记录的详细信息
- **请求头**: Authorization: Bearer {token}
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "recordId": "string",
    "exerciseId": "string",
    "title": "string",
    "typeName": "string",
    "modeName": "string",
    "levelName": "string",
    "score": "number",
    "correctCount": "number",
    "totalCount": "number",
    "correctRate": "number",
    "usedTime": "number",
    "createTime": "string",
    "problems": [
      {
        "problemId": "string",
        "content": "string",
        "userAnswer": "string",
        "correctAnswer": "string",
        "isCorrect": "boolean",
        "usedTime": "number"
      }
    ]
  }
}
```

## 手写作业批改模块接口

### 1. 上传手写作业
- **接口**: POST /homework/upload
- **描述**: 上传手写作业图片
- **请求头**: Authorization: Bearer {token}
- **请求参数**: 表单提交 (multipart/form-data)
```
image: file        // 手写作业图片文件
homeworkType: string // 作业类型：daily(日常作业)/test(测试卷)
description: string  // 可选，作业描述
```
- **响应**:
```json
{
  "code": 200,
  "message": "上传成功",
  "data": {
    "homeworkId": "string",
    "imageUrl": "string",
    "status": "enum"   // 状态：pending(处理中)/completed(已完成)/failed(失败)
  }
}
```

### 2. 获取作业批改结果
- **接口**: GET /homework/{homeworkId}
- **描述**: 获取已上传作业的批改结果
- **请求头**: Authorization: Bearer {token}
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "homeworkId": "string",
    "imageUrl": "string",
    "status": "enum",
    "score": "number",        // 总分
    "correctCount": "number", // 正确题数
    "totalCount": "number",   // 总题数
    "correctRate": "number",  // 正确率
    "problems": [
      {
        "problemId": "string",
        "problemArea": {      // 题目区域坐标
          "x": "number",
          "y": "number", 
          "width": "number",
          "height": "number"
        },
        "content": "string",  // OCR识别的题目内容
        "userAnswer": "string", // OCR识别的用户答案
        "correctAnswer": "string", // 正确答案
        "isCorrect": "boolean"
      }
    ],
    "suggestions": [
      {
        "type": "string",     // 建议类型
        "content": "string"   // 建议内容
      }
    ]
  }
}
```

### 3. 获取作业记录列表
- **接口**: GET /homework/records
- **描述**: 获取用户的作业记录列表
- **请求头**: Authorization: Bearer {token}
- **请求参数**:
```
page: number       // 页码，默认1
pageSize: number   // 每页条数，默认10
status: string     // 可选，筛选特定状态
startDate: string  // 可选，开始日期，格式YYYY-MM-DD
endDate: string    // 可选，结束日期，格式YYYY-MM-DD
```
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "total": "number",
    "page": "number",
    "pageSize": "number",
    "records": [
      {
        "homeworkId": "string",
        "imageUrl": "string",
        "homeworkType": "string",
        "description": "string",
        "status": "enum",
        "score": "number",
        "correctRate": "number",
        "createTime": "string"
      }
    ]
  }
}
```

## 速算小老师模块接口

### 1. 获取题目解答
- **接口**: POST /tutor/solve
- **描述**: 获取特定题目的解答步骤
- **请求头**: Authorization: Bearer {token}
- **请求参数**:
```json
{
  "problem": "string",   // 题目内容，如"125÷5="
  "grade": "number",     // 可选，用户年级（不传则使用用户档案中的年级）
  "detail": "boolean"    // 是否需要详细步骤，默认true
}
```
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "problem": "string",
    "answer": "string",
    "steps": [
      {
        "step": "number",     // 步骤序号
        "description": "string", // 步骤描述
        "formula": "string"   // 公式/算式
      }
    ],
    "explanation": "string",  // 方法解释
    "tips": "string"          // 速算技巧提示
  }
}
```

### 2. 获取知识点讲解
- **接口**: POST /tutor/explain
- **描述**: 获取特定数学知识点的讲解
- **请求头**: Authorization: Bearer {token}
- **请求参数**:
```json
{
  "topic": "string",     // 知识点名称，如"分数乘法"、"小数加法"
  "grade": "number",     // 可选，用户年级（不传则使用用户档案中的年级）
  "withExamples": "boolean" // 是否包含例题，默认true
}
```
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "topic": "string",
    "content": "string",     // 知识点讲解内容
    "keyPoints": [           // 关键要点
      "string"
    ],
    "examples": [            // 例题
      {
        "problem": "string",
        "answer": "string",
        "steps": [
          {
            "step": "number",
            "description": "string",
            "formula": "string"
          }
        ]
      }
    ],
    "relatedTopics": [       // 相关知识点
      {
        "topic": "string",
        "description": "string"
      }
    ]
  }
}
```

### 3. 获取速算技巧
- **接口**: GET /tutor/technique/{techniqueId}
- **描述**: 获取特定速算技巧的详细说明
- **请求头**: Authorization: Bearer {token}
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "techniqueId": "string",
    "name": "string",        // 技巧名称，如"凑十法"
    "description": "string", // 技巧描述
    "applicableScenarios": [ // 适用场景
      "string"
    ],
    "steps": [               // 使用步骤
      {
        "step": "number",
        "description": "string"
      }
    ],
    "examples": [            // 应用例子
      {
        "problem": "string",
        "solution": "string",
        "explanation": "string"
      }
    ],
    "practiceProblems": [    // 练习题目
      {
        "problemId": "string",
        "content": "string"
      }
    ]
  }
}
```

### 4. 获取所有速算技巧列表
- **接口**: GET /tutor/techniques
- **描述**: 获取所有可用的速算技巧列表
- **请求头**: Authorization: Bearer {token}
- **请求参数**:
```
grade: number       // 可选，按年级筛选
operationType: string // 可选，按运算类型筛选（加减乘除）
```
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "techniqueId": "string",
      "name": "string",
      "shortDescription": "string",
      "applicableGrades": [1, 2, 3], // 适用年级
      "operationType": "string"      // 运算类型
    }
  ]
}
```

### 5. 智能问答
- **接口**: POST /tutor/chat
- **描述**: 与速算小老师进行对话，获取回答
- **请求头**: Authorization: Bearer {token}
- **请求参数**:
```json
{
  "message": "string",      // 用户问题内容
  "conversationId": "string" // 可选，对话ID，用于连续对话
}
```
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "conversationId": "string", // 对话ID
    "reply": "string",          // 回答内容
    "relatedResources": [       // 相关资源
      {
        "type": "enum",         // 资源类型：technique/topic/example
        "id": "string",         // 资源ID
        "title": "string",      // 资源标题
        "description": "string" // 资源描述
      }
    ]
  }
}
```

## 天梯排名系统接口

### 1. 获取排行榜
- **接口**: GET /ranking/leaderboard
- **描述**: 获取指定类型的排行榜
- **请求头**: Authorization: Bearer {token}
- **请求参数**:
```
type: string       // 排行类型：daily(日榜)/weekly(周榜)/monthly(月榜)/total(总榜)
category: string   // 排名类别：speed(速度)/accuracy(准确率)/comprehensive(综合)
grade: number      // 可选，按年级筛选
limit: number      // 返回数量限制，默认50
```
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "type": "string",
    "category": "string",
    "updateTime": "string",  // 更新时间
    "userRank": {            // 当前用户排名情况
      "rank": "number",
      "score": "number",
      "change": "number"     // 排名变化，正数表示上升，负数表示下降
    },
    "rankings": [
      {
        "rank": "number",
        "userId": "string",
        "username": "string",
        "avatar": "string",
        "grade": "number",
        "score": "number"    // 根据category不同，可能是平均速度、正确率或综合分数
      }
    ]
  }
}
```

### 2. 获取成就列表
- **接口**: GET /ranking/achievements
- **描述**: 获取所有可获得的成就列表
- **请求头**: Authorization: Bearer {token}
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": [
    {
      "achievementId": "string",
      "name": "string",
      "description": "string",
      "icon": "string",      // 成就图标URL
      "condition": "string", // 获取条件描述
      "category": "string",  // 成就类别
      "isUnlocked": "boolean", // 当前用户是否已解锁
      "progress": {          // 进度情况
        "current": "number",
        "target": "number",
        "percentage": "number"
      }
    }
  ]
}
```

### 3. 获取用户成就
- **接口**: GET /ranking/user-achievements
- **描述**: 获取当前用户已获得的成就
- **请求头**: Authorization: Bearer {token}
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "totalAchievements": "number", // 总成就数
    "unlockedAchievements": "number", // 已解锁成就数
    "latestAchievements": [         // 最近获得的成就
      {
        "achievementId": "string",
        "name": "string",
        "description": "string",
        "icon": "string",
        "unlockTime": "string"
      }
    ],
    "achievements": [
      {
        "achievementId": "string",
        "name": "string",
        "description": "string",
        "icon": "string",
        "category": "string",
        "unlockTime": "string"      // 解锁时间
      }
    ]
  }
}
```

### 4. 获取学习统计数据
- **接口**: GET /ranking/stats
- **描述**: 获取用户学习数据统计
- **请求头**: Authorization: Bearer {token}
- **请求参数**:
```
period: string    // 统计周期：week/month/year/all，默认all
```
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "period": "string",
    "summary": {
      "totalProblems": "number",   // 总题目数
      "correctProblems": "number", // 正确题目数
      "avgCorrectRate": "number",  // 平均正确率
      "avgSpeed": "number",        // 平均速度（题/分钟）
      "totalTime": "number",       // 总学习时间（分钟）
      "longestStreak": "number"    // 最长连续学习天数
    },
    "trends": {                    // 趋势数据，根据period不同粒度不同
      "labels": ["string"],        // 时间标签
      "correctRate": ["number"],   // 正确率数据点
      "speed": ["number"],         // 速度数据点
      "time": ["number"]           // 学习时间数据点
    },
    "problemTypeStats": [          // 题型统计
      {
        "type": "string",          // 题型
        "count": "number",         // 题目数
        "correctRate": "number"    // 正确率
      }
    ]
  }
}
```

## 错题本接口

### 1. 获取错题本列表
- **接口**: GET /mistakes/list
- **描述**: 获取错题本列表
- **请求头**: Authorization: Bearer {token}
- **请求参数**:
```
page: number       // 页码，默认1
pageSize: number   // 每页条数，默认20
category: string   // 可选，题目类别
sortBy: string     // 可选，排序方式：time(时间)/difficulty(难度)
```
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "total": "number",
    "page": "number",
    "pageSize": "number",
    "mistakes": [
      {
        "mistakeId": "string",
        "problem": "string",      // 题目内容
        "correctAnswer": "string", // 正确答案
        "userAnswers": [           // 用户答案历史
          {
            "answer": "string",
            "time": "string",
            "isCorrect": "boolean"
          }
        ],
        "category": "string",     // 题目类别
        "difficulty": "number",   // 难度系数
        "createTime": "string",   // 首次加入时间
        "lastWrongTime": "string" // 最近回答错误时间
      }
    ]
  }
}
```

### 2. 获取错题详情
- **接口**: GET /mistakes/{mistakeId}
- **描述**: 获取错题详细信息
- **请求头**: Authorization: Bearer {token}
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "mistakeId": "string",
    "problem": "string",
    "correctAnswer": "string",
    "userAnswers": [
      {
        "answer": "string",
        "time": "string",
        "sourceType": "string",  // 来源类型：exercise/homework
        "sourceId": "string",    // 来源ID
        "isCorrect": "boolean"
      }
    ],
    "category": "string",
    "relatedKnowledge": [        // 相关知识点
      {
        "topic": "string",
        "description": "string"
      }
    ],
    "solution": {                // 解题方法
      "steps": [
        {
          "step": "number",
          "description": "string",
          "formula": "string"
        }
      ],
      "tips": "string"
    },
    "similarProblems": [         // 相似题目
      {
        "problemId": "string",
        "content": "string",
        "difficulty": "number"
      }
    ]
  }
}
```

### 3. 练习错题
- **接口**: POST /mistakes/practice
- **描述**: 生成基于错题的练习
- **请求头**: Authorization: Bearer {token}
- **请求参数**:
```json
{
  "mistakeIds": ["string"],   // 可选，指定错题ID列表
  "count": "number",          // 题目数量，默认10
  "includeVariants": "boolean" // 是否包含变形题，默认true
}
```
- **响应**:
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "practiceId": "string",
    "title": "错题练习",
    "problems": [
      {
        "problemId": "string",
        "content": "string",
        "originMistakeId": "string", // 原错题ID，变形题为null
        "isVariant": "boolean"      // 是否为变形题
      }
    ]
  }
}
```

### 4. 提交错题练习
- **接口**: POST /mistakes/practice/submit
- **描述**: 提交错题练习答案
- **请求头**: Authorization: Bearer {token}
- **请求参数**:
```json
{
  "practiceId": "string",
  "answers": [
    {
      "problemId": "string",
      "answer": "string"
    }
  ]
}
```
- **响应**:
```json
{
  "code": 200,
  "message": "提交成功",
  "data": {
    "practiceId": "string",
    "correctCount": "number",
    "totalCount": "number",
    "correctRate": "number",
    "masteredMistakes": ["string"], // 已掌握的错题ID列表
    "needReviewMistakes": ["string"] // 需要继续复习的错题ID列表
  }
}
```

## 状态码和错误信息

### 通用错误码
- 400: 请求参数错误
- 401: 未授权（未登录或token失效）
- 403: 权限不足
- 404: 资源不存在
- 500: 服务器内部错误

### 业务错误码
- 1001: 用户名已存在
- 1002: 邮箱已注册
- 1003: 密码不符合规则
- 1004: 旧密码错误
- 2001: 练习类型不存在
- 2002: 题目生成失败
- 3001: 图片上传失败
- 3002: OCR识别失败
- 4001: AI模型调用失败
- 4002: 请求过于频繁
- 5001: 用户排名数据不存在

## 注意事项

1. 文档中所有涉及日期/时间的字符串格式均为ISO 8601标准: `YYYY-MM-DDThh:mm:ss.sssZ`
2. 所有需要身份验证的接口，必须在请求头中包含有效的JWT令牌
3. 所有上传图片的接口默认支持jpg、jpeg、png格式，单张图片大小不超过10MB
4. 数据安全性:
   - 所有API请求应使用HTTPS
   - 敏感数据传输时应进行加密
   - 用户密码存储采用不可逆加密算法
5. 接口升级和兼容性:
   - API版本变更将通过URL中的版本号标识
   - 当前文档描述的是v1版本API
   - 重大变更将提前通知，并保持旧版本API一段时间
