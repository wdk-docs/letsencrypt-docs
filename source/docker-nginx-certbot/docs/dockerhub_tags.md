# 可用的图像标签

`latest`标签总是构建[master branch][master-branch]的头，所以如果可以的话请使用一个更具体的头，因为 master 不应该被认为是"stable"。

`2.0.0`之后的所有标签都是为以下架构构建的:

- linux/amd64
- linux/386 (:warning: not available for [Alpine][alpine-i386] since Nginx `v1.21.0`)
- linux/arm64
- linux/arm/v7 (:warning: not available for Alpine since [tag `v3.1.2`][alpine-armv7])

并且可以将`-alpine`附加到`2.0.1`中的任何标签以获得一个基于 Alpine 的图像。
较不特定的标记将随着较特定的标记的更新而移动。

| Major | Minor | Patch | Nginx              |
| ----: | ----: | ----: | :----------------- |
|     3 |   3.2 | 3.2.2 | 3.2.2-nginx1.23.3  |
|       |       |       | 3.2.2-nginx1.23.2  |
|       |       | 3.2.1 | 3.2.1-nginx1.23.2  |
|       |       |       | 3.2.1-nginx1.23.1  |
|       |       | 3.2.0 | 3.2.0-nginx1.23.1  |
|       |       |       | 3.2.0-nginx1.23.0  |
|       |   3.1 | 3.1.3 | 3.1.3-nginx1.23.0  |
|       |       | 3.1.2 | 3.1.2-nginx1.23.0  |
|       |       |       | 3.1.2-nginx1.21.6  |
|       |       | 3.1.1 | 3.1.1-nginx1.21.6  |
|       |       | 3.1.0 | 3.1.0-nginx1.21.6  |
|       |   3.0 | 3.0.1 | 3.0.1-nginx1.21.6  |
|       |       |       | 3.0.1-nginx1.21.5  |
|       |       |       | 3.0.1-nginx1.21.4  |
|       |       |       | 3.0.1-nginx1.21.3  |
|       |       | 3.0.0 | 3.0.0-nginx1.21.3  |
|     2 |   2.4 | 2.4.1 | 2.4.1-nginx1.21.3  |
|       |       |       | 2.4.1-nginx1.21.1  |
|       |       |       | 2.4.1-nginx1.21.0  |
|       |       | 2.4.0 | 2.4.0-nginx1.21.0  |
|       |   2.3 | 2.3.0 | 2.3.0-nginx1.21.0  |
|       |   2.2 | 2.2.0 | 2.2.0-nginx1.21.0  |
|       |       |       | 2.2.0-nginx1.19.10 |
|       |   2.1 | 2.1.0 | 2.1.0-nginx1.19.10 |
|       |   2.0 | 2.0.1 | 2.0.1-nginx1.19.10 |
|       |       | 2.0.0 | 2.0.0-nginx1.19.10 |
|       |       | 1.3.0 | 1.3.0-nginx1.19.10 |
|       |       |       | 1.3.0-nginx1.19.9  |
|       |       | 1.2.0 | 1.2.0-nginx1.19.9  |
|       |       |       | 1.2.0-nginx1.19.8  |
|       |       | 1.1.0 | 1.1.0-nginx1.19.8  |
|       |       |       | 1.1.0-nginx1.19.7  |
|       |       | 1.0.0 | 1.0.0-nginx1.19.7  |

[master-branch]: https://github.com/JonasAlfredsson/docker-nginx-certbot/tree/master
[alpine-i386]: https://github.com/JonasAlfredsson/docker-nginx-certbot/issues/77
[alpine-armv7]: https://github.com/JonasAlfredsson/docker-nginx-certbot/commit/3fc2d64d3f20aa2163598e57e59a95a79cde1f37
