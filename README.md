# Steins-Gate
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
