---
layout: page
title: Architectural Overview
author: "Daeho Han"
permalink: /
tags: netty architecture
---

## Chapter 2. Architectural Overview
![netty architecture](https://opendevelopergroup.github.io/assets/daeho/netty/architecture.png){:.center-image}

�̹� é�Ϳ�����, Netty���� �����ϴ� core ����� �˾ƺ��� ��� �� ��ɵ��� �Ϻ��� ��Ʈ��ũ ���ø����̼� ���߿� ������ �ִ��� ������ �� �Դϴ�.

### 2.1 Rich Buffer Data Structure
Netty�� �������� byte�� ǥ���ϴµ� *NIO ByteBuffer* ��ſ� �ڽŸ��� buffer API�� ����մϴ�. �̰��� *ByteBuffer*�� ����ϴ� �� ���� Ȯ���� ������ �����ϴ�.  
Netty�� ���ο� buffer Ÿ���� *ChannelBuffer* �� *ByteBuffer*�� ������ �ذ��ϰ� ��Ʈ��ũ ���ø����̼� �������� �䱸������ ������Ű�� ���� ������ �Ǿ����ϴ�. �Ʒ��� ��� Ư¡���� �����Ǿ��ֽ��ϴ�.

> �ʿ��ϴٸ� �ڽŸ��� buffer�� ������ �� �ִ�.  
> Transpartne zero copy is achieved by a built-in composite buffer type.  
> *StringBuffer*ó�� capacity�� �䱸�� ���� Ȯ��Ǵ� ������ buffer Ÿ���� �����ȴ�  
> flip()�� �� �̻� call�� �ʿ䰡 ����.  
> ByteBuffer���� �� �����⵵ �ϴ�.  
