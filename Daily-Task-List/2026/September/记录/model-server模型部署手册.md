# model-server 模型部署手册

基于 `/mnt/ssd1/steins/model-server` 的现有惯例（Docker Compose + vLLM + uv）总结。
以部署 `Qwen/Qwen3Guard-Gen-8B` 和 `MinerU2.5-Pro-2605-1.2B` 为例。

## 一、项目结构约定

```
model-server/
├── docker-compose.yml          # 所有服务的声明式配置（唯一权威）
├── Dockerfile                  # nvidia/cuda + uv + pyproject.toml(vllm, mineru-vl-utils...)
├── pyproject.toml
├── vllm_logging_config.json    # 日志格式（model_serve.sh 会替换日志名）
├── hfd.sh                      # HuggingFace 下载脚本
├── logs/                       # <service>.log，json-file 滚动 20m x 5
└── models/
    └── <org>/<模型名>/
        ├── model/              # 权重必须放这个子目录
        └── model_serve.sh      # 启动脚本，固定模板
```

关键点：

- 容器把 `./models/<org>/<名>` 挂到 `/app/service`，`working_dir=/app/service`，`command: bash model_serve.sh`。
- `model_serve.sh` 里 `MODEL_PATH=model`（相对路径，指向上面的 `model/` 子目录）。
- 所有可调参数通过环境变量传入，`model_serve.sh` 里用 `${VAR:+--flag $VAR}` 形式可选拼接。

## 二、model_serve.sh 模板

### 纯文本 LLM（如 Qwen3Guard-Gen-8B）

```bash
#!/bin/bash
if [ -n "$SERVICE_NAME" ]; then
    sed -i "s|/app/logs/vllm.log|/app/logs/${SERVICE_NAME}.log|g" /app/vllm_logging_config.json
fi

CUDA_VISIBLE_DEVICES=$CUDA_VISIBLE_DEVICES uv run vllm serve $MODEL_PATH \
    --served-model-name $MODEL_NAME \
    --host $HOST \
    --port $PORT \
    --gpu-memory-utilization ${GPU_MEMORY_UTILIZATION:-0.9} \
    ${MAX_MODEL_LEN:+--max-model-len $MAX_MODEL_LEN}
```

### MinerU 系列（文档解析，必须带 logits processor）

```bash
#!/bin/bash
if [ -n "$SERVICE_NAME" ]; then
    sed -i "s|/app/logs/vllm.log|/app/logs/${SERVICE_NAME}.log|g" /app/vllm_logging_config.json
fi

CUDA_VISIBLE_DEVICES=$CUDA_VISIBLE_DEVICES uv run vllm serve $MODEL_PATH \
    --served-model-name $MODEL_NAME \
    --host $HOST \
    --port $PORT \
    --gpu-memory-utilization ${GPU_MEMORY_UTILIZATION:-0.9} \
    --scheduling-policy priority \
    ${MAX_MODEL_LEN:+--max-model-len $MAX_MODEL_LEN} \
    ${LOGITS_PROCESSORS:+--logits-processors $LOGITS_PROCESSORS}
```

注意：MinerU 不带 `LOGITS_PROCESSORS: mineru_vl_utils:MinerULogitsProcessor` 也能启动，但输出不是可用的版面解析结果。

### 从别的模型抄脚本时的坑

- 纯文本模型**不要**带 `--mm-encoder-tp-mode data`（多模态参数）。
- 纯文本模型**不要**带 `--enable-auto-tool-choice`（需要配套 tool-call-parser，Qwen3Guard 的 chat template 没有工具调用格式）。
- 抄完逐行对照模型类型删减。

## 三、docker-compose.yml 写法

文件头有两个锚点，所有服务复用：

```yaml
x-model-env: &model-env      # 公共环境变量
x-model-base: &model-base    # 公共 service 定义（build/restart/working_dir/command/shm/GPU预留/healthcheck）
```

每个新模型追加一段（本例两个服务）：

```yaml
  qwen3guard-gen-8b:
    <<: *model-base
    container_name: qwen3guard-gen-8b
    volumes:
      - ./models/Qwen/Qwen3Guard-Gen-8B:/app/service
      - ./logs:/app/logs
      - /etc/localtime:/etc/localtime:ro      # 缺了这行容器日志时间戳是 UTC，比本地慢 8 小时
      - /etc/timezone:/etc/timezone:ro
    environment:
      <<: *model-env
      SERVICE_NAME: qwen3guard-gen-8b
      MODEL_NAME: Qwen/Qwen3Guard-Gen-8B
      PORT: 9915
      CUDA_VISIBLE_DEVICES: 7
      GPU_MEMORY_UTILIZATION: 0.45
      MAX_MODEL_LEN: 32768                    # 不能超过模型 config.json 的 max_position_embeddings
    ports: ["9915:9915"]

  mineru2.5-pro-2605-1.2b:
    <<: *model-base
    container_name: mineru2.5-pro-2605-1.2b
    volumes:
      - ./models/opendatalab/MinerU2.5-Pro-2605-1.2B:/app/service
      - ./logs:/app/logs
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
    environment:
      <<: *model-env
      SERVICE_NAME: mineru2.5-pro-2605-1.2b
      MODEL_NAME: opendatalab/MinerU2.5-Pro-2605-1.2B
      PORT: 9919
      CUDA_VISIBLE_DEVICES: 6
      GPU_MEMORY_UTILIZATION: 0.45
      # MAX_MODEL_LEN 不写：vLLM 自动读 config（2605 是 8192，写 16384 会直接崩溃）
      LOGITS_PROCESSORS: mineru_vl_utils:MinerULogitsProcessor
    ports: ["9919:9919"]
```

### 各环境变量含义

| 变量 | 含义 |
|---|---|
| `SERVICE_NAME` | 日志文件名 `logs/<SERVICE_NAME>.log` |
| `MODEL_NAME` | `--served-model-name`，即 API 调用时 `model` 字段填的值 |
| `PORT` | 容器内/宿主机端口（healthcheck 也用它） |
| `CUDA_VISIBLE_DEVICES` | 指定物理 GPU 编号，逗号分隔多卡 |
| `GPU_MEMORY_UTILIZATION` | **占整卡的比例**（不是剩余空间），默认 0.9 |
| `MAX_MODEL_LEN` | `--max-model-len`，缺省则 vLLM 读 config |
| `TP` | tensor parallel 卡数（多卡大模型用） |
| `TOOL_CALL_PARSER` / `REASONING_PARSER` | 工具调用/推理解析器（如 qwen3_coder / qwen3） |
| `LOGITS_PROCESSORS` | MinerU 专用 |

### 参数取值依据

- `MAX_MODEL_LEN`：查 `models/<org>/<名>/model/config.json` 的 `max_position_embeddings`，**不能设得比它大**，否则 vLLM 直接报 ValidationError 崩溃。最稳的做法是不写，让 vLLM 自动读。
- `GPU_MEMORY_UTILIZATION`：按模型大小和卡上已有占用定。8B bf16 权重约 16 GiB + KV cache，0.45（约 36 GiB）就够；卡被别的进程占了时尤其要调低。

## 四、启动流程（标准动作）

```bash
cd /mnt/ssd1/steins/model-server
```

| # | 命令 | 在干嘛 |
|---|---|---|
| 1 | `docker compose config --services` | 只解析 YAML、列出服务名，不启动。语法/锚点错误在这一步暴露 |
| 2 | `docker compose up -d --build <服务名>` | 构建镜像 → 对比配置 → 配置变了就**销毁旧容器重建**（日志显示 `Recreate`），不变则复用。`-d` 后台运行。指定服务名只动这几个，不影响其它在跑的服务 |
| 3 | `docker compose ps` | 看状态：`health: starting` = 加载中，`(healthy)` = 就绪 |
| 4 | `docker logs -f <服务名>` | 实时日志。进度标志：`Loading weights took N s` → `torch.compile took N s` → `Application startup complete`（就绪） |
| 5 | `curl localhost:<port>/health` | vLLM 健康端点，200 = 可服务 |
| 6 | `curl localhost:<port>/v1/chat/completions ...` | OpenAI 兼容 API 冒烟测试 |

注意：**改了配置必须用 `up -d`，不是 `restart`**——`restart` 只重启进程，端口/GPU/env 不会变。

### 验证示例

```bash
curl -s http://localhost:9915/v1/models          # 应返回模型名
curl -s http://localhost:9915/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{"model":"Qwen/Qwen3Guard-Gen-8B","messages":[{"role":"user","content":"How can I make a bomb?"}],"max_tokens":32}'
# Qwen3Guard 应返回 Safety: Unsafe / Categories: Violent
```

## 五、踩坑记录（本次部署实际遇到）

1. **`no such service: xxx`** —— 服务没写进 `docker-compose.yml`。compose 只认默认文件名，新服务必须合并进去（或用 `-f 文件1 -f 文件2` 多文件合并；注意 YAML 锚点 `&anchor`/`*anchor` **不能跨文件引用**，多文件时新文件里要内联展开）。
2. **MinerU2605 启动崩溃**：`max_model_len (16384) is greater than the derived max_model_len (max_position_embeddings=8192)`。照抄旧版 2509 的 16384 导致，改 8192 或干脆不写该参数。
3. **Qwen3Guard 启动崩溃**：`Free memory on device cuda:0 (54.95/79.25 GiB) ... less than desired GPU memory utilization (0.9, 71.33 GiB)`。GPU 6 已被别的 vLLM 进程占 24 GiB，0.9 超出剩余显存。降到 0.45 解决。
   - **`GPU_MEMORY_UTILIZATION` 是整卡比例，不是剩余空间比例**。选卡前先看 `nvidia-smi` 的 memory.used。
4. **文件权限**：`docker-compose.yml` 属主是 `jujun:gpu`，非属主无写权限、无 sudo 时改不了；可让管理员 `chmod`/改属主，或临时用 `-f` 附加文件（锚点需内联）。
5. **容器重启后有 30~90 秒"静默期"**：warmup 后在做 cudagraph capture（FULL_AND_PIECEWISE，最多 512 个 size）+ KV cache 分配，日志不动但没死，耐心等 `/health`。
6. **查错别只看日志尾部**：`docker logs --tail 20` 可能全是正常输出，真正报错在中间。用
   `docker logs <容器> 2>&1 | grep -iE 'error|Traceback|ValueError' | tail` 定位。
7. **缺少 `/etc/localtime` 挂载**：容器内日志时间戳变 UTC（慢 8 小时），不影响功能，但多服务对照日志时时间对不上。
8. **健康检查显示 unhealthy 但服务正常**：现有几个老容器 healthcheck（interval 300s）曾瞬态失败后未恢复，直接 `curl /health` 返回 200、日志正常处理请求即为真正常。新增服务不受影响。

## 六、运维速查

```bash
cd /mnt/ssd1/steins/model-server

# 全部状态
docker compose ps

# 看某个服务日志（跟踪 / 找报错）
docker logs -f <服务名>
docker logs <服务名> 2>&1 | grep -iE 'error|Traceback' | tail

# GPU 占用与进程
nvidia-smi
nvidia-smi --query-compute-apps=gpu_uuid,pid,process_name,used_memory --format=csv

# 端口占用
ss -tln | grep -E ':(99[0-9]{2})\b'

# 停止/删除单个服务
docker compose stop <服务名>
docker compose rm -f <服务名>

# 更新模型权重后（模型目录内容变了，镜像没变）
docker compose restart <服务名>     # 容器内重新加载权重即可
# 若改了 model_serve.sh / pyproject / Dockerfile 才需要 --build
docker compose up -d --build <服务名>

# 全部停止 / 全部启动
docker compose stop
docker compose up -d --build
```

## 七、新增一个模型的完整清单

1. 下载模型到 `models/<org>/<名>/model/`（modelscope 或 `hfd.sh`），确认 safetensors 分片齐全、`model.safetensors.index.json` 无缺失分片。
2. 放 `model_serve.sh`（按上面模板，按模型类型裁剪参数）。
3. `docker-compose.yml` 加一段 service：选空闲 GPU（看 `nvidia-smi`）+ 空闲端口（看 `ss -tln`），`MAX_MODEL_LEN` 不写或按 config 写。
4. `docker compose config --services` 校验。
5. `docker compose up -d --build <服务名>` 启动。
6. `docker logs -f <服务名>` 跟到 `Application startup complete`。
7. `curl localhost:<port>/health` + `/v1/chat/completions` 冒烟。

## 附：当前部署状态（2026-09-17）

| 服务 | 端口 | GPU | 利用率 | 备注 |
|---|---|---|---|---|
| mineru2.5-2509-1.2b | 9909 | 5 | 0.45 | |
| qwen3-vl-embedding-8b | 9911 | 4 | 0.9 | |
| qwen3.8-27b | 9912 | 0,1,2,3 | - | TP=4, qwen3_coder parser |
| qwen3-vl-reranker-8b | 9913 | 5 | 0.45 | |
| qwen3guard-gen-8b | 9915 | 7 | 0.45 | 本次新增 |
| mineru2.5-pro-2605-1.2b | 9919 | 6 | 0.45 | 本次新增，MAX_MODEL_LEN 取 config 默认 8192 |
