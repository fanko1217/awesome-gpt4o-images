# 🔌 API 集成指南

## 📊 API 成本对比（100张图片）

| 服务商 | 单张成本 | 100张成本 | 质量 | 速度 | 推荐度 |
|--------|---------|-----------|------|------|--------|
| **Stable Diffusion** | ~$0.002 | **$0.20** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ 最推荐 |
| **通义万相** | ~¥0.008 | **¥0.80** ($0.11) | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ 国内首选 |
| **文心一格** | ~¥0.01 | **¥1.00** ($0.14) | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **豆包** | 待定 | 待定 | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **DALL-E 3** | $0.04 | **$4.00** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ 太贵 |

**30分钟视频示例（30个场景）：**
- Stable Diffusion: $0.06
- 通义万相: ¥0.24 (~$0.03)
- DALL-E 3: $1.20

---

## 1. 通义万相（阿里云）⭐⭐⭐⭐⭐ 最推荐

### 优势
- ✅ 价格便宜（¥0.008/张）
- ✅ 速度快
- ✅ 中文支持好
- ✅ 质量稳定
- ✅ 适合国内用户

### 获取API Key

1. **注册阿里云账号**
   - 访问：https://www.aliyun.com/
   - 注册并完成实名认证

2. **开通通义万相服务**
   - 访问：https://dashscope.console.aliyun.com/
   - 开通"灵积模型服务"
   - 选择"通义万相"文生图服务

3. **创建API Key**
   - 进入控制台：https://dashscope.console.aliyun.com/apiKey
   - 点击"创建新的API-KEY"
   - 保存 `API Key`

4. **充值**
   - 最低充值 ¥10
   - 可生成约 1250 张图片

### 代码集成

```javascript
async function generateWithTongyi(prompt) {
    const response = await fetch('https://dashscope.aliyuncs.com/api/v1/services/aigc/text2image/image-synthesis', {
        method: 'POST',
        headers: {
            'Authorization': `Bearer ${state.apiKey}`,
            'Content-Type': 'application/json',
            'X-DashScope-Async': 'enable'
        },
        body: JSON.stringify({
            model: 'wanx-v1',
            input: {
                prompt: prompt
            },
            parameters: {
                size: '1024*1024',
                n: 1
            }
        })
    });

    const result = await response.json();

    if (result.output && result.output.task_status === 'PENDING') {
        // 轮询获取结果
        const taskId = result.output.task_id;
        return await pollTongyiTask(taskId);
    }

    throw new Error('生成失败');
}

async function pollTongyiTask(taskId) {
    const maxAttempts = 60;
    for (let i = 0; i < maxAttempts; i++) {
        await new Promise(resolve => setTimeout(resolve, 1000));

        const response = await fetch(`https://dashscope.aliyuncs.com/api/v1/tasks/${taskId}`, {
            headers: {
                'Authorization': `Bearer ${state.apiKey}`
            }
        });

        const result = await response.json();

        if (result.output.task_status === 'SUCCEEDED') {
            return result.output.results[0].url;
        } else if (result.output.task_status === 'FAILED') {
            throw new Error('生成失败');
        }
    }

    throw new Error('生成超时');
}
```

### 定价
- 按量计费：¥0.008/张
- 包月套餐：¥99/月（约15000张）

---

## 2. Stable Diffusion API ⭐⭐⭐⭐⭐ 最便宜

### 优势
- ✅ 超低成本（$0.002/张）
- ✅ 开源生态丰富
- ✅ 可自定义模型
- ✅ 国际通用

### 推荐服务商

#### A. Replicate (推荐)

1. **注册账号**
   - 访问：https://replicate.com/
   - 使用 GitHub 账号登录

2. **获取 API Token**
   - 访问：https://replicate.com/account/api-tokens
   - 创建新的 Token

3. **充值**
   - 最低 $10
   - 可生成约 5000 张图片

**代码示例：**
```javascript
async function generateWithSD(prompt) {
    const response = await fetch('https://api.replicate.com/v1/predictions', {
        method: 'POST',
        headers: {
            'Authorization': `Token ${state.apiKey}`,
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            version: 'stability-ai/sdxl:39ed52f2a78e934b3ba6e2a89f5b1c712de7dfea535525255b1aa35c5565e08b',
            input: {
                prompt: prompt,
                width: 1024,
                height: 1024,
                num_outputs: 1
            }
        })
    });

    const prediction = await response.json();

    // 轮询结果
    return await pollReplicatePrediction(prediction.id);
}

async function pollReplicatePrediction(id) {
    const maxAttempts = 60;
    for (let i = 0; i < maxAttempts; i++) {
        await new Promise(resolve => setTimeout(resolve, 1000));

        const response = await fetch(`https://api.replicate.com/v1/predictions/${id}`, {
            headers: {
                'Authorization': `Token ${state.apiKey}`
            }
        });

        const result = await response.json();

        if (result.status === 'succeeded') {
            return result.output[0];
        } else if (result.status === 'failed') {
            throw new Error('生成失败');
        }
    }

    throw new Error('生成超时');
}
```

#### B. Stability AI (官方)

1. **注册账号**
   - 访问：https://platform.stability.ai/

2. **获取 API Key**
   - 创建 API Key

3. **定价**
   - $10 可生成约 10000 张图片

**代码示例：**
```javascript
async function generateWithStabilityAI(prompt) {
    const response = await fetch('https://api.stability.ai/v1/generation/stable-diffusion-xl-1024-v1-0/text-to-image', {
        method: 'POST',
        headers: {
            'Authorization': `Bearer ${state.apiKey}`,
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            text_prompts: [{
                text: prompt,
                weight: 1
            }],
            cfg_scale: 7,
            height: 1024,
            width: 1024,
            samples: 1,
            steps: 30
        })
    });

    const result = await response.json();
    const base64Image = result.artifacts[0].base64;

    // 转换为 blob URL
    const blob = base64ToBlob(base64Image, 'image/png');
    return URL.createObjectURL(blob);
}

function base64ToBlob(base64, mimeType) {
    const byteCharacters = atob(base64);
    const byteNumbers = new Array(byteCharacters.length);
    for (let i = 0; i < byteCharacters.length; i++) {
        byteNumbers[i] = byteCharacters.charCodeAt(i);
    }
    const byteArray = new Uint8Array(byteNumbers);
    return new Blob([byteArray], { type: mimeType });
}
```

---

## 3. 文心一格（百度）⭐⭐⭐⭐

### 优势
- ✅ 中文理解好
- ✅ 百度生态
- ✅ 稳定可靠

### 获取API Key

1. **注册百度智能云**
   - 访问：https://cloud.baidu.com/
   - 注册并实名认证

2. **开通文心一格**
   - 访问：https://console.bce.baidu.com/qianfan/ais/console/applicationConsole/application
   - 创建应用

3. **获取密钥**
   - 获取 `API Key` 和 `Secret Key`

### 代码集成

```javascript
// 第一步：获取 Access Token
async function getBaiduAccessToken() {
    const response = await fetch(
        `https://aip.baidubce.com/oauth/2.0/token?grant_type=client_credentials&client_id=${state.apiKey}&client_secret=${state.apiSecret}`,
        { method: 'POST' }
    );

    const result = await response.json();
    return result.access_token;
}

// 第二步：生成图片
async function generateWithWenxin(prompt) {
    const accessToken = await getBaiduAccessToken();

    const response = await fetch(
        `https://aip.baidubce.com/rpc/2.0/ai_custom/v1/wenxinworkshop/text2image/sd_xl?access_token=${accessToken}`,
        {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                prompt: prompt,
                size: '1024x1024',
                n: 1,
                steps: 20,
                sampler_index: 'Euler a'
            })
        }
    );

    const result = await response.json();

    if (result.data && result.data[0]) {
        // Base64 转 URL
        const base64Image = result.data[0].b64_image;
        const blob = base64ToBlob(base64Image, 'image/png');
        return URL.createObjectURL(blob);
    }

    throw new Error('生成失败');
}
```

### 定价
- 按量计费：¥0.01/张
- 套餐包：¥199/月（约 20000张）

---

## 4. 豆包（字节跳动）⭐⭐⭐

### 状态
- ⚠️ API 接口尚未完全开放
- 建议关注官方公告

### 预计获取方式

1. **访问火山引擎**
   - https://www.volcengine.com/

2. **申请豆包服务**
   - 等待开通

### 临时方案
可以使用字节跳动的其他AI服务，或等待豆包API正式开放。

---

## 5. DALL-E 3（OpenAI）⭐⭐ 质量最高但太贵

### 优势
- ✅ 质量最高
- ✅ 理解能力强
- ❌ 价格昂贵

### 获取 API Key

1. **注册 OpenAI**
   - 访问：https://platform.openai.com/

2. **创建 API Key**
   - https://platform.openai.com/api-keys

3. **充值**
   - 最低 $5

### 代码集成

```javascript
async function generateWithDALLE(prompt) {
    const response = await fetch('https://api.openai.com/v1/images/generations', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${state.apiKey}`
        },
        body: JSON.stringify({
            model: 'dall-e-3',
            prompt: prompt,
            n: 1,
            size: '1792x1024',
            quality: 'hd'
        })
    });

    const result = await response.json();
    return result.data[0].url;
}
```

### 定价
- HD质量：$0.04/张
- 标准质量：$0.02/张

---

## 💡 推荐配置方案

### 方案 A：最省钱（国际用户）
```
文稿分析：GPT-4o (OpenAI)
图片生成：Stable Diffusion (Replicate)

30分钟视频成本：
- 分析：$0.01
- 图片（30张）：$0.06
总计：约 $0.07
```

### 方案 B：最省钱（国内用户）
```
文稿分析：通义千问 (阿里云)
图片生成：通义万相 (阿里云)

30分钟视频成本：
- 分析：¥0.01
- 图片（30张）：¥0.24
总计：约 ¥0.25 ($0.035)
```

### 方案 C：最高质量
```
文稿分析：GPT-4o (OpenAI)
图片生成：DALL-E 3 (OpenAI)

30分钟视频成本：
- 分析：$0.01
- 图片（30张）：$1.20
总计：约 $1.21
```

### 方案 D：平衡方案
```
文稿分析：GPT-4o (OpenAI)
图片生成：通义万相 (阿里云)

30分钟视频成本：
- 分析：$0.01
- 图片（30张）：¥0.24 ($0.03)
总计：约 $0.04
```

---

## 🔧 完整代码实现

将以下代码替换到 `broll-generator-v2.html` 中对应的函数：

```javascript
// 通义万相完整实现
async function generateWithTongyi(prompt) {
    try {
        const response = await fetch('https://dashscope.aliyuncs.com/api/v1/services/aigc/text2image/image-synthesis', {
            method: 'POST',
            headers: {
                'Authorization': `Bearer ${state.apiKey}`,
                'Content-Type': 'application/json',
                'X-DashScope-Async': 'enable'
            },
            body: JSON.stringify({
                model: 'wanx-v1',
                input: {
                    prompt: prompt
                },
                parameters: {
                    size: '1024*1024',
                    n: 1
                }
            })
        });

        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${await response.text()}`);
        }

        const result = await response.json();

        if (result.output && result.output.task_id) {
            return await pollTongyiTask(result.output.task_id);
        }

        throw new Error('任务创建失败');
    } catch (error) {
        console.error('通义万相生成错误:', error);
        throw error;
    }
}

async function pollTongyiTask(taskId) {
    const maxAttempts = 60;
    const pollInterval = 1000;

    for (let i = 0; i < maxAttempts; i++) {
        await new Promise(resolve => setTimeout(resolve, pollInterval));

        try {
            const response = await fetch(`https://dashscope.aliyuncs.com/api/v1/tasks/${taskId}`, {
                headers: {
                    'Authorization': `Bearer ${state.apiKey}`
                }
            });

            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }

            const result = await response.json();

            if (result.output.task_status === 'SUCCEEDED') {
                return result.output.results[0].url;
            } else if (result.output.task_status === 'FAILED') {
                throw new Error(result.output.message || '生成失败');
            }
            // 继续轮询
        } catch (error) {
            console.error(`轮询错误 (尝试 ${i + 1}/${maxAttempts}):`, error);
            if (i === maxAttempts - 1) throw error;
        }
    }

    throw new Error('生成超时，请稍后重试');
}

// 文心一格完整实现
async function generateWithWenxin(prompt) {
    try {
        // 获取 Access Token
        const tokenResponse = await fetch(
            `https://aip.baidubce.com/oauth/2.0/token?grant_type=client_credentials&client_id=${state.apiKey}&client_secret=${state.apiSecret}`,
            { method: 'POST' }
        );

        const tokenResult = await tokenResponse.json();
        const accessToken = tokenResult.access_token;

        // 生成图片
        const response = await fetch(
            `https://aip.baidubce.com/rpc/2.0/ai_custom/v1/wenxinworkshop/text2image/sd_xl?access_token=${accessToken}`,
            {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    prompt: prompt,
                    size: '1024x1024',
                    n: 1,
                    steps: 20,
                    sampler_index: 'Euler a'
                })
            }
        );

        const result = await response.json();

        if (result.data && result.data[0]) {
            const base64Image = result.data[0].b64_image;
            const blob = base64ToBlob(base64Image, 'image/png');
            return URL.createObjectURL(blob);
        }

        throw new Error(result.error_msg || '生成失败');
    } catch (error) {
        console.error('文心一格生成错误:', error);
        throw error;
    }
}

// Base64 转 Blob 工具函数
function base64ToBlob(base64, mimeType) {
    const byteCharacters = atob(base64);
    const byteNumbers = new Array(byteCharacters.length);
    for (let i = 0; i < byteCharacters.length; i++) {
        byteNumbers[i] = byteCharacters.charCodeAt(i);
    }
    const byteArray = new Uint8Array(byteNumbers);
    return new Blob([byteArray], { type: mimeType });
}

// Stable Diffusion (Replicate) 完整实现
async function generateWithSD(prompt) {
    try {
        const response = await fetch('https://api.replicate.com/v1/predictions', {
            method: 'POST',
            headers: {
                'Authorization': `Token ${state.apiKey}`,
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                version: 'stability-ai/sdxl:39ed52f2a78e934b3ba6e2a89f5b1c712de7dfea535525255b1aa35c5565e08b',
                input: {
                    prompt: prompt,
                    width: 1024,
                    height: 1024,
                    num_outputs: 1,
                    scheduler: 'K_EULER',
                    num_inference_steps: 25
                }
            })
        });

        const prediction = await response.json();

        if (prediction.id) {
            return await pollReplicatePrediction(prediction.id);
        }

        throw new Error('预测创建失败');
    } catch (error) {
        console.error('SD生成错误:', error);
        throw error;
    }
}

async function pollReplicatePrediction(id) {
    const maxAttempts = 60;
    const pollInterval = 1000;

    for (let i = 0; i < maxAttempts; i++) {
        await new Promise(resolve => setTimeout(resolve, pollInterval));

        try {
            const response = await fetch(`https://api.replicate.com/v1/predictions/${id}`, {
                headers: {
                    'Authorization': `Token ${state.apiKey}`
                }
            });

            const result = await response.json();

            if (result.status === 'succeeded') {
                return result.output[0];
            } else if (result.status === 'failed') {
                throw new Error(result.error || '生成失败');
            }
        } catch (error) {
            console.error(`轮询错误 (尝试 ${i + 1}/${maxAttempts}):`, error);
            if (i === maxAttempts - 1) throw error;
        }
    }

    throw new Error('生成超时');
}
```

---

## 📝 使用建议

1. **开发测试阶段**
   - 使用 DALL-E 3，质量高便于测试

2. **小规模生产（<10个视频/天）**
   - 使用通义万相，性价比高

3. **大规模生产（>10个视频/天）**
   - 使用 Stable Diffusion，成本最低

4. **追求极致质量**
   - 使用 DALL-E 3，但成本较高

---

## ⚠️ 注意事项

1. **API限流**
   - 大部分服务有并发限制
   - 建议串行生成或控制并发数

2. **错误处理**
   - 添加重试逻辑
   - 记录失败的图片索引

3. **成本控制**
   - 测试时使用小场景数
   - 生产时批量购买套餐

4. **合规性**
   - 确保提示词符合各平台内容政策
   - 避免生成违规内容

---

## 🆘 故障排除

### 通义万相常见问题

**Q: 401 Unauthorized**
- 检查 API Key 是否正确
- 确认是否开通了服务

**Q: 生成超时**
- 增加轮询次数
- 检查网络连接

### Stable Diffusion 常见问题

**Q: 403 Forbidden**
- 检查 Token 格式
- 确认余额充足

**Q: 生成速度慢**
- 减少 steps 参数
- 使用更快的模型

---

需要帮助？欢迎提交 Issue！
