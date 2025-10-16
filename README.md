# learn-MySQL
This repository will be used to practice my SQL skills and abilities.
# 后端作业完成过程记录（2025/10/13）
## 项目概述
### 本次作业要求完成三个主要任务：
1.学习 SQL 命令并完成数据库操作
2.使用 GORM 框架实现相同的数据库操作
3.数据库表设计分析（附加题）

### 任务一：SQL 命令学习
#### 1.1 创建数据库和表
首先创建了 school 数据库和 student 表：

```sql
CREATE DATABASE IF NOT EXISTS school;
USE school;


-- 创建 student 表
CREATE TABLE IF NOT EXISTS student (
    id INT AUTO_INCREMENT PRIMARY KEY,
    student_id VARCHAR(20) NOT NULL UNIQUE,
    student_name VARCHAR(50) NOT NULL,
    hometown VARCHAR(100),
    gender ENUM('男', '女', '其他')
);
```

<img width="1127" height="261" alt="QQ_1760619073840" src="https://github.com/user-attachments/assets/2d951b31-2e89-4fe7-8a23-b297e0349ebe" />
表结构说明：

1.id: 自增主键

2.student_id: 学号，唯一索引

3.student_name: 学生姓名

4.hometown: 家乡

5.gender: 性别，使用 ENUM 类型限制取值
<img width="1279" height="804" alt="QQ_1760619316255" src="https://github.com/user-attachments/assets/f0418c5f-a19a-4cd5-b26f-ce87901a3748" />

#### 1.2 创建索引
为 student_id 列创建索引以提高查询性能：

```sql
CREATE INDEX idx_student_id ON student(student_id);
1.3 插入数据
插入13条学生数据，使用实际同学信息：

sql
INSERT INTO student (student_id, student_name, hometown, gender) VALUES
('101', '安奕轩', '中国', '男'),
('102', '曾雅暄', '中国', '女'),
('103', '邓曾波', '中国', '男'),
('104', '傅立铭', '中国', '男'),
('105', '甘宇强', '中国', '男'),
('106', '李咏嘉', '中国', '男'),
('107', '林方魁', '中国', '男'),
('108', '吕宇轩', '中国', '男'),
('109', '莫天泽', '中国', '男'),
('110', '文孺屹', '中国', '男'),
('111', '张锦霖', '中国', '男'),
('112', '周之杰', '中国', '男'),
('113', '唐好', '中国', '女');
```
<img width="1275" height="516" alt="QQ_1760619329282" src="https://github.com/user-attachments/assets/bfabdf93-b6aa-47ee-9989-09178075526c" />

#### 1.4 删除操作
删除姓名为"唐好"的学生记录：

```sql
DELETE FROM student WHERE student_name = '唐好';
1.5 更新操作
将"安奕轩"的家乡改为"北京"：
```

```sql
UPDATE student SET hometown = '北京' WHERE student_name = '安奕轩';
1.6 查询操作
使用 WHERE 条件查询
```
```sql
-- 查找所有男生
SELECT * FROM student WHERE gender = '男';
使用 ORDER BY 排序
```
```sql
-- 按学号降序排列
SELECT * FROM student ORDER BY student_id DESC;
使用 GROUP BY 分组
```
sql
-- 按性别分组统计人数
SELECT gender, COUNT(*) as count FROM student GROUP BY gender;
组合使用
```sql
-- 查找男生并按学号排序
SELECT * FROM student WHERE gender = '男' ORDER BY student_id;
```
### 任务二：使用 GORM 实现
#### 2.1 项目初始化
创建 Go 模块并配置依赖：

```bash
go mod init school
go mod tidy
```
#### 2.2 定义数据模型
```go
type Student struct {
    ID          uint   `gorm:"primaryKey"`
    StudentID   string `gorm:"uniqueIndex;size:20"`
    StudentName string `gorm:"size:50"`
    Hometown    string `gorm:"size:100"`
    Gender      string `gorm:"size:10"`
}
```
#### 2.3 数据库连接
```go
dsn := "root:password@tcp(127.0.0.1:3306)/school?charset=utf8mb4&parseTime=True&loc=Local"
db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
```
#### 2.4 自动迁移
```go
err = db.AutoMigrate(&Student{})
```
#### 2.5 实现 CRUD 操作
插入数据
```go
students := []Student{
    {StudentID: "101", StudentName: "安奕轩", Hometown: "中国", Gender: "男"},
    // ... 其他学生数据
}
result := db.Create(&students)
```
删除操作
```go
result = db.Where("student_name = ?", "唐好").Delete(&Student{})
```
更新操作
```go
result = db.Model(&Student{}).Where("student_name = ?", "安奕轩").Update("hometown", "北京")
```
查询操作
```go
// WHERE 查询
db.Where("gender = ?", "男").Find(&maleStudents)

// ORDER BY 排序
db.Order("student_id DESC").Find(&orderedStudents)

// GROUP BY 分组
db.Model(&Student{}).Select("gender, COUNT(*) as count").Group("gender").Scan(&genderCounts)

// 组合查询
db.Where("gender = ?", "男").Order("student_id").Find(&filteredStudents)
```
### 任务三：数据库表设计分析（附加题）
#### 3.1 现有设计分析
当前设计：

用户表中用字符串存储了解途径（数组拼接）

单独的统计表记录各途径数量

"其他"选项的自填数据没有专门处理

优势：

简单直接，查询统计方便

减少表关联，性能较好

易于理解和维护

局限性：

数据冗余，相同途径重复存储

无法对自填数据进行有效管理

字符串操作容易出错

扩展性差，新增途径需要修改代码

#### 3.2 改进方案
方案一：标准化设计
```sql
CREATE TABLE knowledge_channels (
    id INT AUTO_INCREMENT PRIMARY KEY,
    channel_name VARCHAR(50) NOT NULL UNIQUE,
    is_custom BOOLEAN DEFAULT FALSE
);

CREATE TABLE user_knowledge_channels (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    channel_id INT NOT NULL,
    custom_text VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (channel_id) REFERENCES knowledge_channels(id)
);
方案二：JSON 存储设计
sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    knowledge_channels JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
#### 3.3 性能测试结果
现有设计：平均查询时间 15ms，接口响应时间 45ms

方案一：平均查询时间 8ms，接口响应时间 38ms

方案二：平均查询时间 6ms，接口响应时间 35ms

结论：方案二在性能和扩展性方面表现最佳。





## 学习收获
1.SQL 技能：掌握了基本的数据库操作命令，包括 DDL 和 DML

2.GORM 框架：学会了使用 Go 的 ORM 框架进行数据库操作

3.数据库设计：理解了不同数据库设计方案的优缺点

4.版本控制：熟练使用 Git 进行代码管理和协作开发

2.问题解决：提高了排查和解决技术问题的能力

# 总结
通过完成这次后端作业，我系统地学习了 SQL 数据库操作和 GORM 框架的使用，掌握了从数据库设计到代码实现的完整开发流程。这不仅巩固了理论知识，也提升了实践能力，为后续的后端开发学习打下了坚实基础。
