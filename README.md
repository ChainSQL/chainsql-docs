# Chainsql-docs

## 环境准备

### 1. 安装python3.7或3.8
### 2. 安装 pip 
1. 从网址 https://pypi.python.org/pypi/pip#downloads 下载最新pip的安装包tar.gz
2. 解压后执行setup.py
3. 找到python的安装路径 ，查看 scripts 文件夹中是否包含 pip相关文件，有则说明安装成功
4. 执行 make.bat html ，会出现各种 module 缺失的错误，需要一个个安装，需要安装的module包括:
- pip install sphinx
- pip install sphinx_rtd_theme
- pip install recommonmark
- pip install pygments_lexer_solidity

### 3. 生成html
#### Windows
```
./make.bat html
```

#### Linux
```
make html
```