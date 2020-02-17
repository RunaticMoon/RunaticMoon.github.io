---
title: "파이썬 명령어 정리"
date: 2020-02-09 18:00 +09:00
categories:
  - Python
tags:
  - 정리
toc: true
---

### pip 업그레이드

```python
# linux
pip install --upgrade pip

# windows
python -m pip install --upgrade pip
```



### Package 설치 및 업그레이드

```python
# 설치
pip install <package_name>

# 업그레이드
pip install <package_name> --upgrade
```



### Package 설치 경로 확인

```python
python -c "import <package_name>; print(<package_name>.__path__)"
```

