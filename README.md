# Steins-Gate
import os
import random
from datetime import datetime
# 定义学生数据类，用于存储和展示学生个体信息
class Student:
    # 初始化方法：接收并存储学生的各项基本属性
    def __init__(self, index, name, gender, cls, sid, college):
        self.index = index # 序号属性
        self.name = name # 姓名属性
        self.gender = gender # 性别属性
        self.cls = cls # 班级属性
        self.sid = sid # 学号属性
        self.college = college # 学院属性
    # 魔术方法：当打印对象或将对象转为字符串时自动调用，返回友好的信息格式
    def __str__(self):
        return f"姓名：{self.name} | 性别：{self.gender} | 班级：{self.cls} | 学号：{self.sid} | 学院：{self.college}"
# 定义考试系统逻辑控制类，封装所有核心业务功能
class ExamSystem:
    # 构造方法：初始化系统，设置文件路径并加载数据
    def __init__(self, file_path):
        self.file_path = file_path # 设置数据源文件路径
        self.students = [] # 用于存放所有Student对象的列表
        self.shuffled_students = [] # 用于存放打乱顺序后的学生列表（生成安排表后使用）
        self.load_data() # 在实例化时自动调用加载数据的方法
    # 静态方法：用于校验输入字符串是否为合法的纯数字格式
    @staticmethod
    def is_valid_number(text):
        return text.isdigit() # 返回布尔值：是数字则为True，否则为False
    # 数据加载方法：读取文本文件并实例化Student对象
    def load_data(self):
        try:
            # 以只读模式和utf-8编码打开学生名单文件
            with open(self.file_path, "r", encoding="utf-8") as f:
                lines = f.readlines()[1:] # 读取所有行并跳过第一行表头
                for line in lines:
                    if line.strip(): # 确保当前行不是只有空格或换行符的空行
                        p = line.split() # 将每一行按空白字符分割成列表
                        # 实例化Student对象并添加到系统列表中
                        self.students.append(Student(p[0], p[1], p[2], p[3], p[4], p[5]))
        except FileNotFoundError:
            print(f"【系统异常】未找到文件：{self.file_path}，请检查路径。")
        except Exception as e:
            print(f"【系统异常】数据加载失败，原因：{e}")
    # 功能1：按学号查找学生信息
    def find_student(self):
        search_id = input("请输入待查找的学生学号：").strip() # 获取用户输入并去除两端空格
        found = False # 初始化查找状态标识
        for s in self.students: # 遍历学生列表
            if s.sid == search_id: # 如果学号匹配
                print(f"【查询结果】{s}") # 打印该学生对象（触发__str__）
                found = True # 标记已找到
                break # 退出循环
        if not found: # 如果循环结束仍未找到
            print("【错误提示】输入的学号不存在，请核对后再试。")
    # 功能2：随机点名功能
    def random_roll_call(self):
        try:
            val = input("请输入需要随机点名的学生数量：") # 获取用户输入
            if not self.is_valid_number(val): # 调用静态方法验证输入
                raise ValueError("点名数量必须是正整数字符。") # 手动触发异常
            count = int(val) # 将字符串转换为整数
            if count &gt; len(self.students): # 检查是否超过总人数
                print(f"【输入错误】数量不能超过总人数（当前总计 {len(self.students)} 人）。")
            elif count &lt;= 0: # 检查是否为非正数
                print("【输入错误】点名数量必须大于0。")
            else:
                # 使用random库的sample方法实现不重复随机采样
                selected = random.sample(self.students, count)
                print(f"--- 随机点名结果（共 {count} 人） ---")
                for i, s in enumerate(selected, 1): # 遍历采样结果并打印
                    print(f"{i}. {s.name} (学号:{s.sid})")
        except ValueError as e:
            print(f"【输入异常】{e}") # 捕获并打印非法数值异常
    # 功能3：生成随机考场安排表
    def generate_exam_table(self):
        if not self.students: # 检查学生列表是否为空
            print("【错误】系统内无学生数据，请先检查数据源。")
            return
        self.shuffled_students = self.students[:] # 浅拷贝一份学生列表
        random.shuffle(self.shuffled_students) # 使用shuffle方法随机打乱列表顺序
        time_str = datetime.now().strftime("%Y-%m-%d %H:%M:%S") # 获取当前格式化时间
        try:
            # 在根目录下创建或覆盖考场安排表文件
            with open("考场安排表.txt", "w", encoding="utf-8") as f:
                f.write(f"生成时间：{time_str}\n") # 写入第一行时间信息
                f.write("座位号\t姓名\t学号\n") # 写入表头
                for i, s in enumerate(self.shuffled_students, 1): # 遍历乱序后的列表
                    f.write(f"{i}\t{s.name}\t{s.sid}\n") # 按格式写入学生座位信息
            print("【操作成功】考场安排表.txt 已在根目录生成。")
        except Exception as e:
            print(f"【文件异常】写入安排表失败：{e}")
    # 功能4：生成独立准考证文件
    def generate_admit_cards(self):
        if not self.shuffled_students: # 检查是否已生成随机安排
            print("【提醒】请先执行功能3以生成考场随机安排顺序。")
            return
        folder = "准考证" # 定义文件夹名称
        try:
            if not os.path.exists(folder): # 如果文件夹不存在
                os.makedirs(folder) # 创建该文件夹
            for i, s in enumerate(self.shuffled_students, 1): # 遍历已排序的学生
                filename = f"{i:02d}.txt" # 格式化文件名，如 01.txt, 02.txt
                file_path = os.path.join(folder, filename) # 拼接完整路径
                with open(file_path, "w", encoding="utf-8") as f: # 创建并写入文件
                    f.write(f"考场座位号：{i:02d}\n") # 写入座位号
                    f.write(f"姓名：{s.name}\n") # 写入姓名
                    f.write(f"学号：{s.sid}\n") # 写入学号
            print(f"【操作成功】已在 '{folder}' 目录下生成 {len(self.shuffled_students)} 份准考证。")
        except Exception as e:
            print(f"【系统异常】准考证生成失败：{e}")
# 主程序运行入口
if __name__ == "__main__":
    # 实例化考试系统，传入学生名单文件名
    system = ExamSystem("人工智能编程语言学生名单.txt")
    while True: # 循环显示菜单
        print("\n" + "="*30)
        print("  人工智能编程语言学生管理系统")
        print("="*30)
        print("1. 查找学生完整信息")
        print("2. 随机点名抽查")
        print("3. 生成随机考场安排表")
        print("4. 批量生成准考证文件")
        print("0. 退出系统")
        print("-" * 30)
        user_choice = input("请选择要执行的操作 (0-4): ").strip() # 获取用户指令
        if user_choice == "1": system.find_student() # 执行查找
        elif user_choice == "2": system.random_roll_call() # 执行点名
        elif user_choice == "3": system.generate_exam_table() # 执行生成表格
        elif user_choice == "4": system.generate_admit_cards() # 执行生成准考证
        elif user_choice == "0": # 退出逻辑
            print("系统已安全关闭，再见！")
            break
        else:
            print("【无效输入】请输入 0 到 4 之间的数字。")
