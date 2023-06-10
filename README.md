# python-project
#简易文本编辑器

import tkinter as tk
import nltk
from nltk.corpus import words
from tkinter import ttk
from docxtpl import DocxTemplate
from tkinter import filedialog
from tkinter import font
from tkinter import colorchooser

def open_file():  #打开文件
    file_path = filedialog.askopenfilename(filetypes=[('文本文件', '.TXT')])
    if file_path:
        with open(file_path, 'r', encoding='GBK') as file:
            text = file.read()
            text_box.delete('1.0', tk.END)
            text_box.insert(tk.END, text)

def save_file():    #保存文件
    file_path = filedialog.asksaveasfilename(defaultextension='*.txt')
    if file_path:
        with open(file_path, 'w', encoding='GB2312') as file:
            text = text_box.get('1.0', tk.END)
            file.write(text)

def set_font_size(size):     #调整字体大小
    global current_font
    current_font.configure(size=size)
    apply_current_font()

def set_font_family(family):    #调整字体
    global current_font
    current_font.configure(family=family)
    apply_current_font()

def apply_current_font():
    text_box.configure(font=current_font)

def toggle_bold():      #文字加粗
    if text_box.tag_ranges(tk.SEL):
        current_tags = text_box.tag_names(tk.SEL_FIRST)
        if "bold" in current_tags:
            text_box.tag_remove("bold", tk.SEL_FIRST, tk.SEL_LAST)
            text_box.tag_configure("bold", font=(current_font['family'], current_font['size'], 'normal'))
        else:
            text_box.tag_add("bold", tk.SEL_FIRST, tk.SEL_LAST)
            text_box.tag_configure("bold", font=(current_font['family'], current_font['size'], 'bold'))


def toggle_underline():     #下划线
    if text_box.tag_ranges(tk.SEL):
        current_tags = text_box.tag_names(tk.SEL_FIRST)
        if "underline" in current_tags:
            text_box.tag_remove("underline", tk.SEL_FIRST, tk.SEL_LAST)
        else:
            text_box.tag_add("underline", tk.SEL_FIRST, tk.SEL_LAST)
            text_box.tag_configure("underline", underline=True)

def set_text_color():
    color = colorchooser.askcolor(title="选择文本颜色")
    if color[1]:
        text_box.tag_configure("colored_text", foreground=color[1])
        text_box.tag_add("colored_text", "sel.first", "sel.last")

def set_alignment(alignment):
    current_tags = text_box.tag_names("sel.first")
    if "alignment" in current_tags:
        text_box.tag_remove("alignment", "sel.first", "sel.last")
    text_box.tag_add("alignment", "sel.first", "sel.last")
    text_box.tag_configure("alignment", justify=alignment)

def align_center():
    set_alignment('center')

def align_left():
    set_alignment('left')

def align_right():
    set_alignment('right')

nltk.download('words')

# 加载单词列表
word_list = set(words.words())

def handle_keypress(event):
    # 检查输入字符是否为空字符串
    if event.char:
        if is_chinese_character(event.char):
            pass
        else:
            autocomplete_english(event)


def is_english_character(char):
    # 判断字符是否为英文字符
    char_code = ord(char)
    return 0x0000 <= char_code <= 0x007F

def is_chinese_character(char):
    # 判断字符是否为中文字符
    char_code = ord(char)
    return 0x4E00 <= char_code <= 0x9FFF

def autocomplete_english(event):
    # 获取当前输入的内容
    current_text = text_box.get("1.0", "end-1c").lower()
    # 获取当前光标所在的位置
    cursor_pos = int(float(text_box.index(tk.INSERT)))
    # 从当前光标位置向前搜索空格或其他分隔符，找到当前单词
    current_word = ""
    for i in range(cursor_pos - 1, 0, -1):
        if current_text[i - 1].isspace():
            break
        current_word = current_text[i - 1] + current_word

    # 根据当前单词和输入的下一个字符，生成自动完成的单词列表
    suggestions = [word for word in word_list if word.startswith(current_word + event.char.lower())]

    if suggestions:
        # 对自动完成的结果进行排序
        suggestions.sort()
        filtered_suggestions = [word for word in suggestions if word.startswith(current_word + event.char.lower())]
        if filtered_suggestions:
            # 创建一个下拉菜单，用于选择自动完成的单词
            # 创建一个下拉菜单，用于选择自动完成的单词
            menu = tk.Menu(text_box, tearoff=0)
            for suggestion in suggestions:
                menu.add_command(label=suggestion, command=lambda s=suggestion: insert_suggestion(s))

            # 显示下拉菜单
            menu.tk_popup(text_box.winfo_rootx(), text_box.winfo_rooty() + text_box.winfo_height())
            menu.grab_release()


def insert_suggestion(suggestion):
    # 将选择的自动完成结果插入到文本框中
    current_word = get_current_word(text_box.get("1.0", "end-1c"))
    suffix = suggestion[len(current_word):]
    text_box.insert("end-1c", suffix)

def get_current_word(text):
    words = text.split()
    if words:
        return words[-1]
    return ""


root = tk.Tk()
root.title("文本编辑器")

text_box = tk.Text(root)
text_box.pack()

current_font = font.Font(size=12)  # 设置默认字体大小为12

menu_bar = tk.Menu(root)
file_menu = tk.Menu(menu_bar, tearoff=0)
file_menu.add_command(label="打开", command=open_file)
file_menu.add_command(label="保存", command=save_file)
file_menu.add_separator()
file_menu.add_command(label="退出", command=root.quit)
menu_bar.add_cascade(label="文件", menu=file_menu)

font_menu = tk.Menu(menu_bar, tearoff=0)
font_size_menu = tk.Menu(font_menu, tearoff=0)
font_size_menu.add_radiobutton(label="12", command=lambda: set_font_size(12))
font_size_menu.add_radiobutton(label="14", command=lambda: set_font_size(14))
font_size_menu.add_radiobutton(label="16", command=lambda: set_font_size(16))
font_menu.add_cascade(label="字体大小", menu=font_size_menu)

font_family_menu = tk.Menu(font_menu, tearoff=0)
font_family_menu.add_radiobutton(label="宋体", command=lambda: set_font_family("宋体"))
font_family_menu.add_radiobutton(label="Times New Roman", command=lambda: set_font_family("Times New Roman"))
font_family_menu.add_radiobutton(label="微软雅黑", command=lambda: set_font_family("微软雅黑"))
font_family_menu.add_radiobutton(label="楷体", command=lambda: set_font_family("楷体"))
font_menu.add_cascade(label="字体样式", menu=font_family_menu)

menu_bar.add_cascade(label="字体", menu=font_menu)

align_menu = tk.Menu(menu_bar, tearoff=0)
align_menu.add_command(label="居中", command=align_center)
align_menu.add_command(label="左对齐", command=align_left)
align_menu.add_command(label="右对齐", command=align_right)
menu_bar.add_cascade(label="段落对齐", menu=align_menu)


edit_menu = tk.Menu(menu_bar, tearoff=0)
edit_menu.add_command(label="加粗", command=toggle_bold)
edit_menu.add_command(label="下划线", command=toggle_underline)
menu_bar.add_cascade(label="编辑", menu=edit_menu)
edit_menu.add_command(label="设置文本颜色", command=set_text_color)

text_box.bind("<Key>", handle_keypress)


root.config(menu=menu_bar)
root.mainloop()


































