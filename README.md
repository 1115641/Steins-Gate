# Steins-Gate
# 陈思阳-25348054-第二次人工智能编程作业
## 1. 任务拆解与 AI 协作策略
步骤1：数据结构化与基础读取。首先要求AI根据提供的文本数据建立基础的读取逻辑，确保能够正确解析制表符分隔的字符串，并将其转化为Python字典或对象。
步骤2：功能模块化实现。将“查找”、“点名”、“生成考场表”和“生成准考证”四个功能分别提问，优先解决逻辑实现（如random.sample的使用和文件写入格式），确保每个功能点独立运行正常。
步骤3：OOP重构与工程规范约束。在功能逻辑跑通后，要求AI将所有代码重构为面向对象（OOP）模式，引入Student类和ExamSystem类，并强制要求加入异常处理和标准库限制，以符合最终作业规范。
## 2. 核心 Prompt 迭代记录
初代 Prompt：编写一个Python程序，读取学生名单文件，实现学号查找和随机点名，最后生成考场安排表和准考证文件。
AI 生成的问题/缺陷：代码采用了纯函数式写法，没有封装类；直接使用了pandas库处理数据；且没有对用户输入非数字字符进行异常捕获，导致输入错误时程序直接崩溃。
优化后的 Prompt (追问)：请使用面向对象编程(OOP)重构代码。要求：1. 定义Student类（含__str__）和ExamSystem类；2. 严禁使用第三方库，仅限os, random, datetime等标准库；3. 必须包含try-except处理文件丢失和输入非法数值的异常；4. 增加一个静态方法用于校验学号或输入格式。
## 3. Debug 与异常处理记录
报错类型/漏洞现象：在生成准考证文件时，程序报错 `FileNotFoundError: [Errno 2] No such file or directory: '准考证/01.txt'`。
解决过程：通过查看报错堆栈发现，虽然代码尝试在“准考证”文件夹下写文件，但如果该文件夹在运行前不存在，Python不会自动创建。我将报错反馈给AI后，AI在代码中加入了 `if not os.path.exists(folder): os.makedirs(folder)` 逻辑，确保了路径的动态生成。
## 4. 人工代码审查 (Code Review)
```python
# 以下为 ExamSystem 类中生成准考证的核心逻辑及人工注释
def generate_admit_cards(self):
    # 检查是否已经生成了随机排序后的名单，如果没有则提醒用户先执行功能3
    if not self.shuffled_students:
        print("【提醒】请先执行功能3以生成考场随机安排顺序。")
        return
    folder = "准考证" # 定义存放准考证的文件夹名称
    try:
        # 检查当前目录下是否存在该文件夹，若不存在则调用os模块创建它
        if not os.path.exists(folder):
            os.makedirs(folder)
        # 遍历打乱顺序后的学生列表，i作为座位号（从1开始），s为学生对象
        for i, s in enumerate(self.shuffled_students, 1):
            filename = f"{i:02d}.txt" # 格式化文件名，确保1显示为01，保持排序整齐
            file_path = os.path.join(folder, filename) # 跨平台路径拼接，确保在不同系统下路径正确
            # 使用with语句安全打开文件，设置utf-8编码防止中文乱码
            with open(file_path, "w", encoding="utf-8") as f:
                f.write(f"考场座位号：{i:02d}\n") # 将座位号写入文件第一行
                f.write(f"姓名：{s.name}\n") # 调用学生对象的name属性写入姓名
                f.write(f"学号：{s.sid}\n") # 调用学生对象的sid属性写入学号
        # 循环结束后，打印成功提示及生成的文件总数
        print(f"【操作成功】已在 '{folder}' 目录下生成 {len(self.shuffled_students)} 份准考证。")
    except Exception as e:
        # 如果在创建文件夹或写文件过程中发生任何IO异常，捕获并打印错误原因
        print(f"【系统异常】准考证生成失败：{e}")
import os
import random
from datetime import datetime
# 初始化学生数据文件，如果不存在则创建（方便直接运行演示）
def init_data_file():
    filename = "人工智能编程语言学生名单.txt"
    if not os.path.exists(filename):
        content = """序号	姓名	性别	班级	学号	学院
1	张三	男	1	2001101	电气
2	李四	女	2	2001102	能动
3	魏五	男	3	2001103	能动
4	丁七	女	1	2001104	电气
5	王八	男	2	2001105	计算机
6	何九	女	3	2001106	计算机
7	赵十	男	1	2001107	外国语
8	吴一	女	2	2001108	外国语
9	蓝二	男	3	2001109	经管
10	刘六	女	1	2001110	经管"""
        with open(filename, "w", encoding="utf-8") as f:
            f.write(content)
# 加载学生信息
def load_students():
    students = []
    try:
        with open("人工智能编程语言学生名单.txt", "r", encoding="utf-8") as f:
            lines = f.readlines()[1:] # 跳过表头
            for line in lines:
                if line.strip():
                    parts = line.split()
                    students.append({
                        "姓名": parts[1],
                        "性别": parts[2],
                        "班级": parts[3],
                        "学号": parts[4],
                        "学院": parts[5]
                    })
    except FileNotFoundError:
        print("错误：未找到学生名单文件！")
    return students
# 功能1：查找学生
def find_student(students):
    search_id = input("请输入要查找的学号: ").strip()
    for s in students:
        if s["学号"] == search_id:
            print(f"查询结果 -&gt; 姓名: {s['姓名']}, 性别: {s['性别']}, 班级: {s['班级']}, 学院: {s['学院']}")
            return
    print("抱歉，未找到该学号的学生信息。")
# 功能2：随机点名
def random_roll_call(students):
    try:
        count_input = input("请输入需要点名的学生数量: ")
        count = int(count_input) # 转换数字
        if count &gt; len(students):
            print(f"错误：点名人数不能超过总人数({len(students)}人)。")
        elif count &lt;= 0:
            print("错误：请输入大于0的数字。")
        else:
            selected = random.sample(students, count)
            print("--- 随机点名名单 ---")
            for i, s in enumerate(selected, 1):
                print(f"{i}. {s['姓名']} ({s['学号']})")
    except ValueError:
        print("异常：请输入有效的数字字符！")
# 功能3：生成考场安排表
def generate_exam_table(students):
    shuffled_students = students[:]
    random.shuffle(shuffled_students) # 随机打乱
    now_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    filename = "考场安排表.txt"
    with open(filename, "w", encoding="utf-8") as f:
        f.write(f"生成时间：{now_time}\n") # 第一行时间
        f.write("考场座位号\t姓名\t学号\n")
        for i, s in enumerate(shuffled_students, 1):
            f.write(f"{i:02d}\t{s['姓名']}\t{s['学号']}\n")
    print(f"成功：已生成『{filename}』")
    return shuffled_students
# 功能4：生成准考证目录
def generate_admit_cards(shuffled_students):
    folder_name = "准考证" # 准考证文件夹名称
    if not os.path.exists(folder_name):
        os.makedirs(folder_name)
    for i, s in enumerate(shuffled_students, 1):
        file_name = f"{i:02d}.txt" # 文件名如 01.txt
        file_path = os.path.join(folder_name, file_name)
        with open(file_path, "w", encoding="utf-8") as f:
            f.write(f"考场座位号: {i:02d}\n")
            f.write(f"姓名: {s['姓名']}\n")
            f.write(f"学号: {s['学号']}\n")
    print(f"成功：已在『{folder_name}』文件夹下生成所有准考证。")
# 主程序入口
def main():
    init_data_file()
    students = load_students()
    if not students: return
    shuffled_cache = [] # 用于缓存打乱后的名单以生成准考证
    while True:
        print("\n=== 学生信息管理系统 ===")
        print("1. 信息查找 (按学号)")
        print("2. 随机点名")
        print("3. 生成考场安排表")
        print("4. 生成准考证文件")
        print("0. 退出系统")
        choice = input("请选择功能 (0-4): ")
        if choice == "1":
            find_student(students)
        elif choice == "2":
            random_roll_call(students)
        elif choice == "3":
            shuffled_cache = generate_exam_table(students)
        elif choice == "4":
            if not shuffled_cache:
                print("提示：请先执行功能3生成考场安排表。")
            else:
                generate_admit_cards(shuffled_cache)
        elif choice == "0":
            print("系统已退出。")
            break
        else:
            print("无效输入，请重新选择。")
if __name__ == "__main__":
    main()
