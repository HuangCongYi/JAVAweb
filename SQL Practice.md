
-- 创建数据库
CREATE DATABASE IF NOT EXISTS employee_management;
USE employee_management;

-- 创建部门表
CREATE TABLE departments (
    dept_id INT PRIMARY KEY AUTO_INCREMENT,
    dept_name VARCHAR(50) NOT NULL,
    location VARCHAR(50)
);

-- 创建员工表
CREATE TABLE employees (
    emp_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    phone_number VARCHAR(20),
    hire_date DATE,
    job_title VARCHAR(50),
    salary DECIMAL(10, 2),
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);

-- 创建项目表
CREATE TABLE projects (
    project_id INT PRIMARY KEY AUTO_INCREMENT,
    project_name VARCHAR(100) NOT NULL,
    start_date DATE,
    end_date DATE
);

-- 创建员工项目关联表
CREATE TABLE employee_projects (
    emp_id INT,
    project_id INT,
    PRIMARY KEY (emp_id, project_id),
    FOREIGN KEY (emp_id) REFERENCES employees(emp_id),
    FOREIGN KEY (project_id) REFERENCES projects(project_id)
);

-- 插入部门数据
INSERT INTO departments (dept_name, location) VALUES
('HR', 'New York'),
('IT', 'San Francisco'),
('Finance', 'Chicago'),
('Marketing', 'Los Angeles'),
('Operations', 'Houston');

-- 插入员工数据
INSERT INTO employees (first_name, last_name, email, phone_number, hire_date, job_title, salary, dept_id) VALUES
('John', 'Doe', 'john.doe@example.com', '1234567890', '2020-01-15', 'HR Manager', 75000.00, 1),
('Jane', 'Smith', 'jane.smith@example.com', '2345678901', '2019-05-20', 'Software Engineer', 85000.00, 2),
('Mike', 'Johnson', 'mike.johnson@example.com', '3456789012', '2018-11-10', 'Financial Analyst', 70000.00, 3),
('Emily', 'Brown', 'emily.brown@example.com', '4567890123', '2021-03-01', 'Marketing Specialist', 65000.00, 4),
('David', 'Wilson', 'david.wilson@example.com', '5678901234', '2017-09-15', 'Operations Manager', 80000.00, 5),
('Sarah', 'Lee', 'sarah.lee@example.com', '6789012345', '2020-07-01', 'IT Support', 60000.00, 2),
('Chris', 'Anderson', 'chris.anderson@example.com', '7890123456', '2019-12-01', 'Accountant', 68000.00, 3),
('Lisa', 'Taylor', 'lisa.taylor@example.com', '8901234567', '2022-01-10', 'HR Assistant', 55000.00, 1),
('Tom', 'Martin', 'tom.martin@example.com', '9012345678', '2018-06-15', 'Software Developer', 82000.00, 2),
('Amy', 'White', 'amy.white@example.com', '0123456789', '2021-09-01', 'Marketing Manager', 78000.00, 4);

-- 插入项目数据
INSERT INTO projects (project_name, start_date, end_date) VALUES
('Website Redesign', '2023-01-01', '2024-11-30'),
('ERP Implementation', '2023-03-15', '2025-03-14'),
('Marketing Campaign', '2023-05-01', '2023-08-31'),
('Financial Audit', '2023-07-01', '2025-09-30'),
('New Product Launch', '2023-09-01', '2024-02-29');

-- 插入员工项目关联数据
INSERT INTO employee_projects (emp_id, project_id) VALUES
(2, 1), (6, 1), (9, 1),
(2, 2), (5, 2), (6, 2), (9, 2),
(4, 3), (10, 3),
(3, 4), (7, 4),
(4, 5), (5, 5), (10, 5);

#员工信息练习题
# 13. 查询即将在半年内到期的项目。
SELECT project_id,project_name
FROM projects
WHERE end_date BETWEEN curdate() AND date_add(curdate(),INTERVAL 6 MONTH );
# 14. 查询至少参与了两个项目的员工。
SELECT ep.emp_id,concat(first_name,' ',last_name) AS fullname,count(project_id) AS project_count
FROM employees e
join employee_projects ep ON e.emp_id = ep.emp_id
GROUP BY ep.emp_id
HAVING count(project_id)>=2;
# 15. 查询没有参与任何项目的员工。
SELECT e.emp_id,concat(first_name,' ',last_name)
FROM employees e
LEFT JOIN employee_projects ep ON e.emp_id = ep.emp_id
WHERE ep.project_id IS NULL;
# 16. 计算每个项目参与的员工数量。
SELECT count(emp_id) AS employ_count,project_id
FROM employee_projects
GROUP BY project_id;
# 17. 查询工资第二高的员工信息。
SELECT emp_id, CONCAT(first_name, ' ', last_name) AS full_name, email, phone_number, hire_date, job_title, salary, dept_id
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
# 18. 查询每个部门工资最高的员工。
SELECT emp_id, CONCAT(first_name, ' ', last_name) AS full_name, email, phone_number, hire_date, job_title, salary, e.dept_id
FROM employees e
JOIN(
    SELECT dept_id,max(salary) AS max_salary
    FROM employees
    GROUP BY dept_id
    ) AS max ON e.dept_id=max.dept_id AND e.salary=max.max_salary;
# 19. 计算每个部门的工资总和,并按照工资总和降序排列。
SELECT dept_id,sum(salary) AS total_salary
FROM employees e
GROUP BY dept_id
ORDER BY total_salary DESC;
# 20. 查询员工姓名、部门名称和工资。
SELECT concat(first_name,' ',last_name),dept_name,salary
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id;
# 21. 查询每个员工的上级主管(假设emp_id小的是上级)。
SELECT e.emp_id,concat(e.first_name,' ',e.last_name) AS employee_name,concat(m.first_name,' ',m.last_name) AS manage_name
FROM employees e
LEFT JOIN employees m ON e.emp_id>m.emp_id;
# 22. 查询所有员工的工作岗位,不要重复。
SELECT DISTINCT e.job_title
FROM employees e;
# 24. 查询工资高于其所在部门平均工资的员工。
SELECT emp_id,dept_id,concat(e.first_name,' ',e.last_name)
FROM employees e
WHERE e.salary>(
    SELECT avg(salary) AS avg_salary
    FROM employees
    WHERE dept_id=e.dept_id
    );
# 25. 查询每个部门工资前两名的员工。
WITH RankedEmployees AS (
    SELECT
        e.first_name,
        e.last_name,
        e.salary,
        d.dept_name,
        ROW_NUMBER() OVER (PARTITION BY d.dept_id ORDER BY e.salary DESC) AS SalaryRank
    FROM
        employees e
    JOIN
        departments d ON e.dept_id = d.dept_id
)
SELECT
    first_name,
    last_name,
    salary,
    dept_name
FROM
    RankedEmployees
WHERE
    SalaryRank <= 2;
# 26. 查询跨部门的项目(参与员工来自不同部门)。
SELECT DISTINCT p.project_name
FROM projects p
JOIN employee_projects ep ON p.project_id = ep.project_id
JOIN employees e1 ON ep.emp_id = e1.emp_id
JOIN employees e2 ON ep.emp_id = e2.emp_id
WHERE e1.dept_id <> e2.dept_id;
# 27. 查询每个员工的工作年限,并按工作年限降序排序。
SELECT
    first_name,
    last_name,
    TIMESTAMPDIFF(YEAR, hire_date, CURDATE()) AS work_experience
FROM
    employees
ORDER BY
    work_experience DESC;
# 28. 查询本月过生日的员工(假设hire_date是生日)。
SELECT
    first_name,
    last_name,
    hire_date
FROM
    employees
WHERE
    MONTH(hire_date) = MONTH(CURDATE()) AND
    DAY(hire_date) = DAY(CURDATE());
# 29. 查询即将在90天内到期的项目和负责该项目的员工。
SELECT
    p.project_name,
    e.first_name,
    e.last_name
FROM
    projects p
JOIN
    employee_projects ep ON p.project_id = ep.project_id
JOIN
    employees e ON ep.emp_id = e.emp_id
WHERE
    p.end_date BETWEEN CURDATE() AND DATE_ADD(CURDATE(), INTERVAL 90 DAY);
# 30. 计算每个项目的持续时间(天数)。
SELECT
    project_name,
    DATEDIFF(end_date, start_date) AS duration_days
FROM
    projects;
# 31. 查询没有进行中项目的部门。
SELECT
    d.dept_name
FROM
    departments d
WHERE
    d.dept_id NOT IN (
        SELECT DISTINCT e.dept_id
        FROM employees e
        JOIN employee_projects ep ON e.emp_id = ep.emp_id
        JOIN projects p ON ep.project_id = p.project_id
        WHERE p.start_date <= CURDATE() AND p.end_date >= CURDATE()
    );
# 32. 查询员工数量最多的部门。
SELECT
    d.dept_name,
    COUNT(e.emp_id) AS employee_count
FROM
    departments d
LEFT JOIN
    employees e ON d.dept_id = e.dept_id
GROUP BY
    d.dept_name
ORDER BY
    employee_count DESC
LIMIT 1;
# 33. 查询参与项目最多的部门。
SELECT
    d.dept_name,
    COUNT(DISTINCT ep.project_id) AS project_count
FROM
    departments d
JOIN
    employees e ON d.dept_id = e.dept_id
JOIN
    employee_projects ep ON e.emp_id = ep.emp_id
GROUP BY
    d.dept_name
ORDER BY
    project_count DESC
LIMIT 1;
# 34. 计算每个员工的薪资涨幅(假设每年涨5%)。
SELECT
    first_name,
    last_name,
    salary,
    salary * POWER(1.05, TIMESTAMPDIFF(YEAR, hire_date, CURDATE())) AS new_salary
FROM
    employees;
# 35. 查询入职时间最长的3名员工。
SELECT
    first_name,
    last_name,
    hire_date
FROM
    employees
ORDER BY
    hire_date ASC
LIMIT 3;
# 36. 查询名字和姓氏相同的员工。
SELECT
    first_name,
    last_name
FROM
    employees
WHERE
    first_name = last_name;
# 37. 查询每个部门薪资最低的员工。
SELECT
    d.dept_name,
    e.first_name,
    e.last_name,
    e.salary
FROM
    departments d
JOIN
    employees e ON d.dept_id = e.dept_id
WHERE
    e.salary = (
        SELECT MIN(salary)
        FROM employees
        WHERE dept_id = d.dept_id
    );
# 38. 查询哪些部门的平均工资高于公司的平均工资。
SELECT
    d.dept_name
FROM
    departments d
JOIN
    employees e ON d.dept_id = e.dept_id
GROUP BY
    d.dept_name
HAVING
    AVG(e.salary) > (SELECT AVG(salary) FROM employees);
# 39. 查询姓名包含"son"的员工信息。
SELECT
    first_name,
    last_name,
    email,
    phone_number,
    hire_date,
    job_title,
    salary
FROM
    employees
WHERE
    first_name LIKE '%son%' OR last_name LIKE '%son%';
# 40. 查询所有员工的工资级别(可以自定义工资级别)。
SELECT
    first_name,
    last_name,
    salary,
    CASE
        WHEN salary < 50000 THEN '低级'
        WHEN salary BETWEEN 50000 AND 70000 THEN '中级'
        WHEN salary BETWEEN 70001 AND 90000 THEN '高级'
        ELSE '顶级'
    END AS salary_level
FROM
    employees;
# 41. 查询每个项目的完成进度(根据当前日期和项目的开始及结束日期)。
SELECT
    project_name,
    start_date,
    end_date,
    CASE
        WHEN start_date > CURDATE() THEN '未开始'
        WHEN end_date < CURDATE() THEN '已完成'
        ELSE CONCAT(ROUND(DATEDIFF(CURDATE(), start_date) / DATEDIFF(end_date, start_date) * 100, 2), '% 完成')
    END AS progress
FROM
    projects;
# 42. 查询每个经理(假设job_title包含'Manager'的都是经理)管理的员工数量。
SELECT
    e.first_name AS manager_first_name,
    e.last_name AS manager_last_name,
    COUNT(sub.emp_id) AS employee_count
FROM
    employees e
LEFT JOIN
    employees sub ON e.emp_id = sub.dept_id
WHERE
    e.job_title LIKE '%Manager%'
GROUP BY
    e.emp_id;
# 43. 查询工作岗位名称里包含"Manager"但不在管理岗位(salary<70000)的员工。
SELECT
    first_name,
    last_name,
    job_title,
    salary
FROM
    employees
WHERE
    job_title LIKE '%Manager%' AND
    salary < 70000;
# 44. 计算每个部门的男女比例(假设以名字首字母A-M为女性,N-Z为男性)。
SELECT
    d.dept_name,
    SUM(CASE WHEN LEFT(first_name, 1) BETWEEN 'A' AND 'M' THEN 1 ELSE 0 END) AS female_count,
    SUM(CASE WHEN LEFT(first_name, 1) BETWEEN 'N' AND 'Z' THEN 1 ELSE 0 END) AS male_count
FROM
    departments d
JOIN
    employees e ON d.dept_id = e.dept_id
GROUP BY
    d.dept_name;
# 45. 查询每个部门年龄最大和最小的员工(假设hire_date反应了年龄)。
SELECT
    d.dept_name,
    MIN(e.hire_date) AS oldest_employee,
    MAX(e.hire_date) AS youngest_employee
FROM
    departments d
JOIN
    employees e ON d.dept_id = e.dept_id
GROUP BY
    d.dept_name;
# 46. 查询连续3天都有员工入职的日期。
SELECT
    hire_date
FROM
    employees
GROUP BY
    hire_date
HAVING
    COUNT(DISTINCT hire_date) >= 3
ORDER BY
    hire_date DESC;
# 47. 查询员工姓名和他参与的项目数量。
SELECT
    e.first_name,
    e.last_name,
    COUNT(ep.project_id) AS project_count
FROM
    employees e
LEFT JOIN
    employee_projects ep ON e.emp_id = ep.emp_id
GROUP BY
    e.emp_id;
# 48. 查询每个部门工资最高的3名员工。
SELECT
    d.dept_name,
    e.first_name,
    e.last_name,
    e.salary
FROM
    employees e
JOIN
    departments d ON e.dept_id = d.dept_id
WHERE
    (d.dept_id, e.salary) IN (
        SELECT
            dept_id, salary
        FROM
            employees
        WHERE
            dept_id = d.dept_id
        ORDER BY
            salary DESC
        LIMIT 3
    );
# 49. 计算每个员工的工资与其所在部门平均工资的差值。
SELECT
    e.first_name,
    e.last_name,
    e.salary,
    AVG(e2.salary) AS dept_avg_salary,
    e.salary - AVG(e2.salary) AS salary_difference
FROM
    employees e
JOIN
    employees e2 ON e.dept_id = e2.dept_id
GROUP BY
    e.emp_id;
# 50. 查询所有项目的信息,包括项目名称、负责人姓名(假设工资最高的为负责人)、开始日期和结束日期。
SELECT
    p.project_name,
    CONCAT(e.first_name, ' ', e.last_name) AS manager_name,
    p.start_date,
    p.end_date
FROM
    projects p
JOIN
    employee_projects ep ON p.project_id = ep.project_id
JOIN
    employees e ON ep.emp_id = e.emp_id
WHERE
    e.salary = (
        SELECT MAX(salary)
        FROM employees
        WHERE emp_id IN (
            SELECT emp_id
            FROM employee_projects
            WHERE project_id = p.project_id
        )
    );

#学生选课题
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for course
-- ----------------------------
DROP TABLE IF EXISTS `course`;
CREATE TABLE `course`  (
  `course_id` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `course_name` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `teacher_id` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL,
  `credits` decimal(2, 1) NULL DEFAULT NULL,
  PRIMARY KEY (`course_id`) USING BTREE,
  INDEX `teacher_id`(`teacher_id`) USING BTREE,
  CONSTRAINT `course_ibfk_1` FOREIGN KEY (`teacher_id`) REFERENCES `teacher` (`teacher_id`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of course
-- ----------------------------
INSERT INTO `course` VALUES ('C001', '高等数学', 'T001', 4.0);
INSERT INTO `course` VALUES ('C002', '大学物理', 'T002', 3.5);
INSERT INTO `course` VALUES ('C003', '程序设计', 'T003', 4.0);
INSERT INTO `course` VALUES ('C004', '数据结构', 'T004', 3.5);
INSERT INTO `course` VALUES ('C005', '数据库原理', 'T005', 4.0);
INSERT INTO `course` VALUES ('C006', '操作系统', 'T006', 3.5);

-- ----------------------------
-- Table structure for score
-- ----------------------------
DROP TABLE IF EXISTS `score`;
CREATE TABLE `score`  (
  `student_id` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `course_id` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `score` decimal(4, 1) NULL DEFAULT NULL,
  PRIMARY KEY (`student_id`, `course_id`) USING BTREE,
  INDEX `course_id`(`course_id`) USING BTREE,
  CONSTRAINT `score_ibfk_1` FOREIGN KEY (`student_id`) REFERENCES `student` (`student_id`) ON DELETE RESTRICT ON UPDATE RESTRICT,
  CONSTRAINT `score_ibfk_2` FOREIGN KEY (`course_id`) REFERENCES `course` (`course_id`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of score
-- ----------------------------
INSERT INTO `score` VALUES ('2021001', 'C001', 85.5);
INSERT INTO `score` VALUES ('2021001', 'C002', 78.0);
INSERT INTO `score` VALUES ('2021001', 'C003', 90.5);
INSERT INTO `score` VALUES ('2021002', 'C001', 92.0);
INSERT INTO `score` VALUES ('2021002', 'C002', 83.5);
INSERT INTO `score` VALUES ('2021002', 'C004', 58.0);
INSERT INTO `score` VALUES ('2021003', 'C001', 76.5);
INSERT INTO `score` VALUES ('2021003', 'C003', 85.0);
INSERT INTO `score` VALUES ('2021003', 'C005', 69.5);
INSERT INTO `score` VALUES ('2021004', 'C002', 88.5);
INSERT INTO `score` VALUES ('2021004', 'C004', 92.5);
INSERT INTO `score` VALUES ('2021004', 'C006', 86.0);
INSERT INTO `score` VALUES ('2021005', 'C001', 61.0);
INSERT INTO `score` VALUES ('2021005', 'C003', 87.5);
INSERT INTO `score` VALUES ('2021005', 'C005', 84.0);
INSERT INTO `score` VALUES ('2021006', 'C002', 79.5);
INSERT INTO `score` VALUES ('2021006', 'C004', 83.0);
INSERT INTO `score` VALUES ('2021006', 'C006', 90.0);
INSERT INTO `score` VALUES ('2021007', 'C001', 93.5);
INSERT INTO `score` VALUES ('2021007', 'C003', 89.0);
INSERT INTO `score` VALUES ('2021007', 'C005', 94.5);
INSERT INTO `score` VALUES ('2021008', 'C002', 86.5);
INSERT INTO `score` VALUES ('2021008', 'C004', 91.0);
INSERT INTO `score` VALUES ('2021008', 'C006', 87.5);
INSERT INTO `score` VALUES ('2021009', 'C001', 80.0);
INSERT INTO `score` VALUES ('2021009', 'C003', 62.5);
INSERT INTO `score` VALUES ('2021009', 'C005', 85.5);
INSERT INTO `score` VALUES ('2021010', 'C002', 64.5);
INSERT INTO `score` VALUES ('2021010', 'C004', 89.5);
INSERT INTO `score` VALUES ('2021010', 'C006', 93.0);

-- ----------------------------
-- Table structure for student
-- ----------------------------
DROP TABLE IF EXISTS `student`;
CREATE TABLE `student`  (
  `student_id` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `name` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `gender` char(1) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `birth_date` date NULL DEFAULT NULL,
  `my_class` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL,
  PRIMARY KEY (`student_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of student
-- ----------------------------
INSERT INTO `student` VALUES ('2021001', '张三', '男', '2003-05-15', '计算机一班');
INSERT INTO `student` VALUES ('2021002', '李四', '女', '2003-08-22', '计算机一班');
INSERT INTO `student` VALUES ('2021003', '王五', '男', '2002-11-30', '数学一班');
INSERT INTO `student` VALUES ('2021004', '赵六', '女', '2003-02-14', '数学一班');
INSERT INTO `student` VALUES ('2021005', '钱七', '男', '2002-07-08', '物理一班');
INSERT INTO `student` VALUES ('2021006', '孙八', '女', '2003-09-19', '物理一班');
INSERT INTO `student` VALUES ('2021007', '周九', '男', '2002-12-01', '化学一班');
INSERT INTO `student` VALUES ('2021008', '吴十', '女', '2003-03-25', '化学一班');
INSERT INTO `student` VALUES ('2021009', '郑十一', '男', '2002-06-11', '生物一班');
INSERT INTO `student` VALUES ('2021010', '王十二', '女', '2003-10-05', '生物一班');

-- ----------------------------
-- Table structure for teacher
-- ----------------------------
DROP TABLE IF EXISTS `teacher`;
CREATE TABLE `teacher`  (
  `teacher_id` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `name` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `gender` char(1) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NOT NULL,
  `birth_date` date NULL DEFAULT NULL,
  `title` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci NULL DEFAULT NULL,
  PRIMARY KEY (`teacher_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_0900_ai_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of teacher
-- ----------------------------
INSERT INTO `teacher` VALUES ('T001', '张教授', '男', '1975-03-12', '教授');
INSERT INTO `teacher` VALUES ('T002', '李副教授', '女', '1980-07-22', '副教授');
INSERT INTO `teacher` VALUES ('T003', '王讲师', '男', '1985-11-08', '讲师');
INSERT INTO `teacher` VALUES ('T004', '赵助教', '女', '1990-05-15', '助教');
INSERT INTO `teacher` VALUES ('T005', '钱教授', '男', '1972-09-30', '教授');
INSERT INTO `teacher` VALUES ('T006', '孙副教授', '女', '1978-12-18', '副教授');
INSERT INTO `teacher` VALUES ('T007', '周讲师', '男', '1983-04-25', '讲师');
INSERT INTO `teacher` VALUES ('T008', '吴助教', '女', '1988-08-07', '助教');
INSERT INTO `teacher` VALUES ('T009', '郑教授', '男', '1970-01-01', '教授');
INSERT INTO `teacher` VALUES ('T010', '刘副教授', '女', '1976-06-14', '副教授');

SET FOREIGN_KEY_CHECKS = 1;

# 1. 查询所有学生的信息。
    SELECT * FROM student;
# 2. 查询所有课程的信息。
SELECT * FROM course;
# 3. 查询所有学生的姓名、学号和班级。
SELECT name, student_id, my_class FROM student;
# 4. 查询所有教师的姓名和职称。
SELECT name, title FROM teacher;
# 5. 查询不同课程的平均分数。
SELECT course_id, AVG(score) AS average_score FROM score GROUP BY course_id;
# 6. 查询每个学生的平均分数。
SELECT student_id, AVG(score) AS average_score FROM score GROUP BY student_id;
# 7. 查询分数大于85分的学生学号和课程号。
SELECT student_id, course_id FROM score WHERE score > 85;
# 8. 查询每门课程的选课人数。
SELECT course_id, COUNT(student_id) AS enrollment_count FROM score GROUP BY course_id;
# 9. 查询选修了"高等数学"课程的学生姓名和分数。
SELECT s.name, sc.score FROM student s
JOIN score sc ON s.student_id = sc.student_id
WHERE sc.course_id = 'C001';
# 10. 查询没有选修"大学物理"课程的学生姓名。
SELECT name FROM student WHERE student_id NOT IN (
    SELECT student_id FROM score WHERE course_id = 'C002'
);
# 11. 查询C001比C002课程成绩高的学生信息及课程分数。
SELECT s.student_id, s.name, sc1.score AS score_C001, sc2.score AS score_C002
FROM student s
JOIN score sc1 ON s.student_id = sc1.student_id AND sc1.course_id = 'C001'
JOIN score sc2 ON s.student_id = sc2.student_id AND sc2.course_id = 'C002'
WHERE sc1.score > sc2.score;
# 12. 统计各科成绩各分数段人数：课程编号，课程名称，[100-85]，[85-70]，[70-60]，[60-0] 及所占百分比
SELECT course_id, course_name,
    SUM(CASE WHEN score BETWEEN 85 AND 100 THEN 1 ELSE 0 END) AS '100-85',
    SUM(CASE WHEN score BETWEEN 70 AND 85 THEN 1 ELSE 0 END) AS '85-70',
    SUM(CASE WHEN score BETWEEN 60 AND 70 THEN 1 ELSE 0 END) AS '70-60',
    SUM(CASE WHEN score < 60 THEN 1 ELSE 0 END) AS '60-0'
FROM score s
JOIN course c ON s.course_id = c.course_id
GROUP BY course_id, course_name;
# 13. 查询选择C002课程但没选择C004课程的成绩情况(不存在时显示为 null )。
SELECT student_id, score FROM score WHERE course_id = 'C002' AND student_id NOT IN (
    SELECT student_id FROM score WHERE course_id = 'C004'
);
# 14. 查询平均分数最高的学生姓名和平均分数。
SELECT s.name, AVG(sc.score) AS average_score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY s.student_id
ORDER BY average_score DESC
LIMIT 1;
# 15. 查询总分最高的前三名学生的姓名和总分。
SELECT s.name, SUM(sc.score) AS total_score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY s.student_id
ORDER BY total_score DESC
LIMIT 3;
# 16. 查询各科成绩最高分、最低分和平均分。要求如下：
# 以如下形式显示：课程 ID，课程 name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率
# 及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90
# 要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列
SELECT c.course_id, c.course_name,
    MAX(s.score) AS highest_score,
    MIN(s.score) AS lowest_score,
    AVG(s.score) AS average_score,
    SUM(CASE WHEN s.score >= 60 THEN 1 ELSE 0 END) / COUNT(s.score) AS pass_rate,
    SUM(CASE WHEN s.score BETWEEN 70 AND 80 THEN 1 ELSE 0 END) / COUNT(s.score) AS medium_rate,
    SUM(CASE WHEN s.score BETWEEN 80 AND 90 THEN 1 ELSE 0 END) / COUNT(s.score) AS good_rate,
    SUM(CASE WHEN s.score >= 90 THEN 1 ELSE 0 END) / COUNT(s.score) AS excellent_rate
FROM score s
JOIN course c ON s.course_id = c.course_id
GROUP BY c.course_id, c.course_name;
# 17. 查询男生和女生的人数。
SELECT gender, COUNT(*) AS count FROM student GROUP BY gender;
# 18. 查询年龄最大的学生姓名。
SELECT name FROM student ORDER BY birth_date ASC LIMIT 1;
# 19. 查询年龄最小的教师姓名。
SELECT name FROM teacher ORDER BY birth_date DESC LIMIT 1;
# 20. 查询学过「张教授」授课的同学的信息。
SELECT s.* FROM student s
JOIN score sc ON s.student_id = sc.student_id
WHERE sc.course_id IN (
    SELECT c.course_id FROM course c WHERE c.teacher_id = 'T001'
);
# 21. 查询查询至少有一门课与学号为"2021001"的同学所学相同的同学的信息 。
SELECT DISTINCT s2.*
FROM score sc1
JOIN score sc2 ON sc1.course_id = sc2.course_id AND sc1.student_id != sc2.student_id
JOIN student s1 ON sc1.student_id = s1.student_id
JOIN student s2 ON sc2.student_id = s2.student_id
WHERE s1.student_id = '2021001';
# 22. 查询每门课程的平均分数，并按平均分数降序排列。
SELECT course_id, AVG(score) AS average_score
FROM score
GROUP BY course_id
ORDER BY average_score DESC;
# 23. 查询学号为"2021001"的学生所有课程的分数。
SELECT course_id, score FROM score WHERE student_id = '2021001';
# 24. 查询所有学生的姓名、选修的课程名称和分数。
SELECT s.name, c.course_name, sc.score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
JOIN course c ON sc.course_id = c.course_id;
# 25. 查询每个教师所教授课程的平均分数。
SELECT t.name AS teacher_name, AVG(sc.score) AS average_score
FROM teacher t
JOIN course c ON t.teacher_id = c.teacher_id
JOIN score sc ON c.course_id = sc.course_id
GROUP BY t.teacher_id;
# 26. 查询分数在80到90之间的学生姓名和课程名称。
SELECT s.name, c.course_name FROM student s
JOIN score sc ON s.student_id = sc.student_id
JOIN course c ON sc.course_id = c.course_id
WHERE sc.score BETWEEN 80 AND 90;
# 27. 查询每个班级的平均分数。
SELECT my_class, AVG(sc.score) AS average_score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY my_class;
# 28. 查询没学过"王讲师"老师讲授的任一门课程的学生姓名。
SELECT name FROM student WHERE student_id NOT IN (
    SELECT student_id FROM score WHERE course_id IN (
        SELECT course_id FROM course WHERE teacher_id = 'T003'
    )
);
# 29. 查询两门及其以上小于85分的同学的学号，姓名及其平均成绩 。
SELECT s.student_id, s.name, AVG(sc.score) AS average_score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
WHERE sc.score < 85
GROUP BY s.student_id
HAVING COUNT(sc.course_id) >= 2;
# 30. 查询所有学生的总分并按降序排列。
SELECT s.student_id, s.name, SUM(sc.score) AS total_score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY s.student_id
ORDER BY total_score DESC;
# 31. 查询平均分数超过85分的课程名称。
SELECT c.course_name
FROM course c
JOIN score sc ON c.course_id = sc.course_id
GROUP BY c.course_id
HAVING AVG(sc.score) > 85;
# 32. 查询每个学生的平均成绩排名。
SELECT s.student_id, name, AVG(score) AS average_score,
       RANK() OVER (ORDER BY AVG(score) DESC) AS `rank`
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY student_id, name;
# 33. 查询每门课程分数最高的学生姓名和分数。
SELECT c.course_name, s.name, sc.score
FROM score sc
JOIN student s ON sc.student_id = s.student_id
JOIN course c ON sc.course_id = c.course_id
WHERE (sc.course_id, sc.score) IN (
    SELECT course_id, MAX(score)
    FROM score
    GROUP BY course_id
);
# 34. 查询选修了"高等数学"和"大学物理"的学生姓名。
SELECT s.name
FROM student s
WHERE s.student_id IN (
    SELECT student_id FROM score WHERE course_id = 'C001'
) AND s.student_id IN (
    SELECT student_id FROM score WHERE course_id = 'C002'
);
# 35. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩（没有选课则为空）。
SELECT s.student_id, s.name, c.course_name, sc.score,
       AVG(sc2.score) OVER (PARTITION BY s.student_id) AS average_score
FROM student s
LEFT JOIN score sc ON s.student_id = sc.student_id
LEFT JOIN course c ON sc.course_id = c.course_id
LEFT JOIN score sc2 ON s.student_id = sc2.student_id;
# 36. 查询分数最高和最低的学生姓名及其分数。
SELECT name, score FROM student s
JOIN score sc ON s.student_id = sc.student_id
WHERE score = (SELECT MAX(score) FROM score)
   OR score = (SELECT MIN(score) FROM score);
# 37. 查询每个班级的最高分和最低分。
SELECT my_class,
       MAX(sc.score) AS highest_score,
       MIN(sc.score) AS lowest_score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY my_class;
# 38. 查询每门课程的优秀率（优秀为90分）。
SELECT c.course_id, c.course_name,
       SUM(CASE WHEN sc.score >= 90 THEN 1 ELSE 0 END) / COUNT(sc.score) AS excellent_rate
FROM score sc
JOIN course c ON sc.course_id = c.course_id
GROUP BY c.course_id, c.course_name;
# 39. 查询平均分数超过班级平均分数的学生。
SELECT s.student_id, s.name
FROM student s
JOIN (
    SELECT my_class, AVG(sc.score) AS class_average
    FROM student s
    JOIN score sc ON s.student_id = sc.student_id
    GROUP BY my_class
) AS class_avg ON s.my_class = class_avg.my_class
JOIN score sc ON s.student_id = sc.student_id
GROUP BY s.student_id
HAVING AVG(sc.score) > class_avg.class_average;
# 40. 查询每个学生的分数及其与课程平均分的差值。
SELECT s.student_id, s.name, sc.course_id, sc.score,
       (sc.score - avg_score) AS score_difference
FROM student s
JOIN score sc ON s.student_id = sc.student_id
JOIN (
    SELECT course_id, AVG(score) AS avg_score
    FROM score
    GROUP BY course_id
) avg_scores ON sc.course_id = avg_scores.course_id;
# 41. 查询至少有一门课程分数低于80分的学生姓名。
SELECT DISTINCT s.name
FROM student s
JOIN score sc ON s.student_id = sc.student_id
WHERE sc.score < 80;
# 42. 查询所有课程分数都高于85分的学生姓名。
SELECT s.name
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY s.student_id, s.name
HAVING MIN(sc.score) > 85;
# 43. 查询查询平均成绩大于等于90分的同学的学生编号和学生姓名和平均成绩。
SELECT s.student_id, s.name, AVG(sc.score) AS average_score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY s.student_id, s.name
HAVING AVG(sc.score) >= 90;
# 44. 查询选修课程数量最少的学生姓名。
SELECT s.name
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY s.student_id, s.name
ORDER BY COUNT(sc.course_id) ASC
LIMIT 1;
# 45. 查询每个班级的第2名学生（按平均分数排名）。
SELECT my_class, student_id, name, AVG(sc.score) AS average_score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY my_class, student_id, name
HAVING RANK() OVER (PARTITION BY my_class ORDER BY AVG(sc.score) DESC) = 2;
# 46. 查询每门课程分数前三名的学生姓名和分数。
SELECT sc.course_id, s.name, sc.score
FROM score sc
JOIN student s ON sc.student_id = s.student_id
WHERE (sc.course_id, sc.score) IN (
    SELECT course_id, score
    FROM score AS sub
    WHERE sub.course_id = sc.course_id
    ORDER BY score DESC
    LIMIT 3
);
# 47. 查询平均分数最高和最低的班级。
SELECT my_class, AVG(sc.score) AS average_score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY my_class
ORDER BY average_score DESC
LIMIT 1; -- 最高分

SELECT my_class, AVG(sc.score) AS average_score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY my_class
ORDER BY average_score ASC
LIMIT 1; -- 最低分
# 48. 查询每个学生的总分和他所在班级的平均分数。
SELECT s.student_id, s.name, SUM(sc.score) AS total_score,
       (SELECT AVG(score) FROM score WHERE student_id IN
           (SELECT student_id FROM student WHERE my_class = s.my_class)) AS class_average
FROM student s
JOIN score sc ON s.student_id = sc.student_id
GROUP BY s.student_id, s.name;
# 49. 查询每个学生的最高分的课程名称, 学生名称，成绩。
SELECT s.name, sc.course_id, sc.score
FROM student s
JOIN score sc ON s.student_id = sc.student_id
WHERE (s.student_id, sc.score) IN (
    SELECT student_id, MAX(score)
    FROM score
    GROUP BY student_id
);
# 50. 查询每个班级的学生人数和平均年龄。
SELECT my_class, COUNT(*) AS student_count,
       AVG(TIMESTAMPDIFF(YEAR, birth_date, CURDATE())) AS average_age
FROM student
GROUP BY my_class;