# docker/backend/Dockerfile (优化版)
# Stage 1: Build the application using Maven
FROM maven:3.8-openjdk-11 AS builder

WORKDIR /app

# 1. 复制 pom.xml 和子模块目录，【优先执行 install】
# 这一步会编译所有模块，并将内部依赖 (ruoyi-common/framework/...) 安装到本地 Maven 仓库
COPY pom.xml .
COPY ruoyi-common ruoyi-common
COPY ruoyi-framework ruoyi-framework
COPY ruoyi-system ruoyi-system
COPY ruoyi-quartz ruoyi-quartz
COPY ruoyi-generator ruoyi-generator
COPY ruoyi-admin ruoyi-admin
RUN mvn clean install -DskipTests -B

# 2. 最终打包
# 这一步会使用已经安装好的内部依赖和下载好的远程依赖来完成 ruoyi-admin 的打包
# 使用 package 目标即可，无需 clean，因为前面已经 clean install 过了
RUN mvn package -pl ruoyi-admin -am -DskipTests

# Stage 2: Create the runtime image
# 使用一个精简的 JRE 镜像
FROM eclipse-temurin:11-jre-alpine

WORKDIR /app

# 从构建阶段复制构建好的 Jar 包
# Jar 包通常在模块的 target 目录下
COPY --from=builder /app/ruoyi-admin/target/ruoyi-admin.jar ./app.jar

# 暴露后端服务端口 (默认 8080)
EXPOSE 8080

# 设置容器启动时执行的命令
# 使用 exec 格式，允许传递信号
# 通过环境变量传递数据库和 Redis 配置 (将在 docker-compose 中设置)
ENTRYPOINT ["java", \
            "-Djava.security.egd=file:/dev/./urandom", \
            "-jar", \
            "app.jar"]