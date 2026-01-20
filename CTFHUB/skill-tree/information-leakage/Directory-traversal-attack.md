## CTFHUB 技能树/WEB/信息泄露/目录遍历
<img width="1808" height="474" alt="屏幕截图 2026-01-20 150450" src="https://github.com/user-attachments/assets/a0eda1e7-1158-4f44-9a7c-c3263a44dac4" />

### 题目：目录遍历
这是web类型下的第一道题，其实很简单，就一直点击目录，找到flag.txt就可以了。以下是我的解题思路：
#### 1.打开对应网址，然后点击绿色按钮（开始寻找flag）
<img width="2519" height="538" alt="屏幕截图 2026-01-20 142324" src="https://github.com/user-attachments/assets/76d26881-72ef-47e4-8f13-cf8d46d04ae7" />

#### 2.进入 /flag_in_here/ 目录后，看到4个子目录：1/, 2/, 3/, 4/。这个题目的关键是需要递归地遍历这些目录结构。每个目录下都有相同的子目录结构（1/, 2/, 3/, 4/）。
<img width="2519" height="642" alt="屏幕截图 2026-01-20 142330" src="https://github.com/user-attachments/assets/ac23886b-1b99-4ae0-961f-abd67b4448a0" />

#### 3.在路径 /flag_in_here/3/3/ 下找到了 flag.txt 文件,点击查看即可
<img width="2519" height="465" alt="屏幕截图 2026-01-20 142427" src="https://github.com/user-attachments/assets/046d8b6a-f108-4c23-b786-8046668c9ab8" />

<img width="2519" height="275" alt="屏幕截图 2026-01-20 142434" src="https://github.com/user-attachments/assets/68d2e2f5-39a5-44ce-81ba-9913bbd698a2" />

#### 4.当然有兴趣的话，也可以用python脚本进行递归寻找flag(使用递归深度优先搜索来遍历所有可能的目录路径),脚本如下：

```python
import requests
from bs4 import BeautifulSoup
import sys
import time

def get_directory_listing(url):
    """获取目录列表页面并解析其中的目录链接"""
    try:
        response = requests.get(url, timeout=10)
        if response.status_code == 200:
            soup = BeautifulSoup(response.text, 'html.parser')
            directories = []
            file_links = []
          
            # 查找所有的链接
            for link in soup.find_all('a'):
                href = link.get('href')
                if href and href != '../':
                    full_url = f"{url.rstrip('/')}/{href}" if not url.endswith('/') else f"{url}{href}"
                    if href.endswith('/'):  # 这是一个目录
                        directories.append(full_url)
                    else:  # 这是一个文件
                        file_links.append(full_url)
          
            return directories, file_links
        return [], []
    except Exception as e:
        print(f"访问 {url} 出错: {e}")
        return [], []

def recursive_search(base_url, max_depth=5, visited=None):
    """递归搜索目录结构"""
    if visited is None:
        visited = set()
  
    # 防止无限递归
    if max_depth <= 0 or base_url in visited:
        return []
  
    print(f"搜索: {base_url}")
    visited.add(base_url)
  
    # 获取当前目录的内容
    directories, files = get_directory_listing(base_url)
  
    results = []
  
    # 检查是否有文件
    for file_url in files:
        if 'flag' in file_url.lower():
            print(f"发现疑似flag文件: {file_url}")
            results.append(file_url)
      
        # 如果是txt文件，也检查内容
        if file_url.lower().endswith('.txt'):
            try:
                file_response = requests.get(file_url, timeout=10)
                if file_response.status_code == 200:
                    content = file_response.text.strip()
                    if 'flag' in content.lower() or 'ctfhub' in content.lower():
                        print(f"找到flag! 文件: {file_url}")
                        print(f"内容: {content[:100]}...")
                        results.append((file_url, content))
            except Exception as e:
                print(f"读取文件 {file_url} 出错: {e}")
  
    # 递归搜索子目录
    for directory in directories:
        if directory not in visited:
            sub_results = recursive_search(directory, max_depth - 1, visited)
            results.extend(sub_results)
  
    return results

def brute_force_search(base_url, path=""):
    """暴力搜索所有可能的4级目录组合（1-4的数字组合）"""
    numbers = ['1', '2', '3', '4']
    found_files = []
  
    # 尝试所有可能的4级目录组合
    for a in numbers:
        print(f"尝试第一级: {a}")
        for b in numbers:
            for c in numbers:
                for d in numbers:
                    # 构建路径
                    test_path = f"{path}/{a}/{b}/{c}/{d}".strip('/')
                    test_url = f"{base_url.rstrip('/')}/{test_path}"
                  
                    # 先尝试访问目录本身
                    try:
                        response = requests.get(test_url, timeout=5)
                        if response.status_code == 200:
                            # 检查是否有文件
                            soup = BeautifulSoup(response.text, 'html.parser')
                            for link in soup.find_all('a'):
                                href = link.get('href')
                                if href and not href.endswith('/'):
                                    file_url = f"{test_url.rstrip('/')}/{href}"
                                    if 'flag' in href.lower() or 'flag' in file_url.lower():
                                        print(f"找到可能包含flag的目录: {test_url}")
                                        print(f"文件: {file_url}")
                                      
                                        # 尝试读取文件
                                        try:
                                            file_response = requests.get(file_url, timeout=5)
                                            if file_response.status_code == 200:
                                                content = file_response.text
                                                if 'ctfhub' in content.lower():
                                                    print(f"找到flag! 文件: {file_url}")
                                                    print(f"内容: {content[:100]}...")
                                                    return file_url, content
                                        except Exception as e:
                                            print(f"读取文件出错: {e}")
                    except Exception as e:
                        continue
  
    return None, None

def main():
    base_url = "http://challenge-62630ba6b132d3ad.sandbox.ctfhub.com:10800/flag_in_here"
  
    print("=" * 60)
    print("CTFHub 目录遍历挑战 - 自动破解脚本")
    print("=" * 60)
    print(f"目标URL: {base_url}")
    print("\n开始搜索...\n")
  
    # 方法1: 递归深度优先搜索
    print("方法1: 递归深度优先搜索")
    print("-" * 40)
    results = recursive_search(base_url)
  
    if results:
        print("\n递归搜索完成！找到以下结果：")
        for result in results:
            if isinstance(result, tuple):
                print(f"文件: {result[0]}")
                print(f"Flag: {result[1]}")
            else:
                print(f"发现文件: {result}")
    else:
        print("\n递归搜索未找到flag，尝试暴力破解...")
      
        # 方法2: 暴力搜索
        print("\n方法2: 暴力搜索所有可能的目录组合")
        print("-" * 40)
        flag_url, flag_content = brute_force_search(base_url)
      
        if flag_url and flag_content:
            print("\n暴力搜索成功！")
            print(f"Flag文件: {flag_url}")
            print(f"Flag内容: {flag_content}")
        else:
            print("\n暴力搜索也未找到flag，可能需要手动检查。")
  
    print("\n" + "=" * 60)
    print("脚本执行完成！")
    print("=" * 60)

if __name__ == "__main__":
    main()
```

## 脚本说明

### 核心功能：

1. **`get_directory_listing(url)`**:
   - 获取指定URL的目录列表
   - 解析HTML页面，提取目录链接和文件链接
   - 区分目录（以`/`结尾）和文件

2. **`recursive_search(base_url, max_depth, visited)`**:
   - 递归深度优先搜索算法
   - 自动跟踪已访问的URL，防止无限循环
   - 检查每个文件的内容，寻找包含"flag"或"ctfhub"的文件
   - 限制搜索深度（默认5层）

3. **`brute_force_search(base_url, path)`**:
   - 暴力搜索所有可能的目录组合
   - 生成所有可能的1-4数字组合（4层深度）
   - 检查每个目录是否存在.txt文件
   - 读取文件内容，查找flag

4. **`main()`**:
   - 主函数，组织整个搜索流程
   - 先尝试递归搜索，如果没找到则尝试暴力搜索

### 使用方式：

```python
# 直接运行脚本
python solve_ctfhub_directory_traversal.py

# 或者修改目标URL
base_url = "http://你的目标URL/flag_in_here"
```

### 脚本输出：

脚本会显示：
1. 正在搜索的目录
2. 发现的文件
3. 找到flag时的详细信息

### 关键设计：

1. **智能过滤**：只关注包含"flag"关键字或.txt扩展名的文件
2. **防重复**：使用集合记录已访问的URL
3. **容错处理**：使用try-except处理网络错误
4. **进度显示**：实时显示搜索进展
5. **双重策略**：递归搜索 + 暴力搜索确保覆盖所有可能

由于重新开了一个靶场，所以flag的位置发送了改变，但是脚本是可以成功执行的，这一题其实很简单的。
<img width="1645" height="936" alt="image" src="https://github.com/user-attachments/assets/294ceaf7-d245-4f74-8629-46ac6ab1eb3a" />
