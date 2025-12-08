# Open-AutoGLM

[中文阅读.](./README.md)

<div align="center">
<img src=resources/logo.svg width="20%"/>
</div>
<p align="center">
    👋 Join our <a href="resources/WECHAT.md" target="_blank">WeChat</a> community
</p>

## Project Introduction

Phone Agent is a mobile intelligent assistant framework built on AutoGLM. It can understand phone screen content in a
multimodal way and help users complete tasks through automated operations. The system controls devices through ADB (
Android Debug Bridge), uses vision-language models for screen perception, and combines intelligent planning capabilities
to generate and execute operation workflows. Users only need to describe their requirements in natural language, such
as "Open Xiaohongshu and search for food", and Phone Agent will automatically parse the intent, understand the current
interface, plan the next action, and complete the entire workflow. The system also has a built-in sensitive operation
confirmation mechanism and supports manual takeover in login or verification code scenarios. Additionally, it provides
remote ADB debugging capabilities, allowing device connection via WiFi or network for flexible remote control and
development.

> ⚠️ This project is for research and learning purposes only. It is strictly prohibited to use it for illegal
> information gathering, system interference, or any illegal activities. Please carefully review
> the [Terms of Use](resources/privacy_policy.txt).

## Model Download Links

| Model            | Download Links                                                                                                                               |
|------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| AutoGLM-Phone-9B | [🤗 Hugging Face](https://huggingface.co/zai-org/AutoGLM-Phone-9B)<br>[🤖 ModelScope](https://modelscope.cn/models/ZhipuAI/AutoGLM-Phone-9B) |

Currently, this model only supports Chinese. A multilingual version of the model is coming soon.

## Environment Setup

### 1. Python Environment

Python 3.10 or above is recommended.

### 2. ADB (Android Debug Bridge)

1. Download the official ADB [installation package](https://developer.android.com/tools/releases/platform-tools?hl=en),
   and extract it to a custom path
2. Configure environment variables

- MacOS configuration method: In `Terminal` or any command-line tool

  ```bash
  # Assuming the extracted directory is ~/Downloads/platform-tools. If not, please adjust the command accordingly.
  export PATH=${PATH}:~/Downloads/platform-tools
  ```

- Windows configuration method: You can refer to
  this [third-party tutorial](https://blog.csdn.net/x2584179909/article/details/108319973) for configuration.

### 3. Android 7.0+ Device or Emulator with `Developer Mode` and `USB Debugging` Enabled

1. Enable Developer Mode: The typical method is to find `Settings > About Phone > Build Number` and tap it quickly about
   10 times until a popup shows "Developer mode enabled". Different phones may vary slightly; if you can't find it,
   search online for a tutorial.
2. Enable USB Debugging: After enabling Developer Mode, go to `Settings > Developer Options > USB Debugging` and enable
   it
3. Some devices may require a restart after enabling developer options for changes to take effect. You can test it:
   connect your phone to the computer via USB cable, then run `adb devices` to check if device information appears. If
   not, the connection has failed.

### 4. Install ADB Keyboard (for text input)

Download the [installation package](https://github.com/senzhk/ADBKeyBoard/blob/master/ADBKeyboard.apk) and install it on
the corresponding Android device.
Note: After installation, you need to enable `ADB Keyboard` in `Settings > Input Method` or `Settings > Keyboard List`
for it to work.

## Deployment Preparation

### 1. Install Dependencies

```bash
pip install -r requirements.txt 
pip install -e .
```

### 2. Configure ADB

Make sure **your USB cable supports data transfer**, not just charging.

Ensure ADB is installed and connect the device with a **USB cable**:

```bash
# Check connected devices
adb devices

# Output should show your device, e.g.:
# List of devices attached
# emulator-5554   device
```

### 3. Start the Model Service

1. Download the model and install the inference engine framework according to the `For Model Deployment` section in
   `requirements.txt`.
2. Start via SGlang / vLLM to get an OpenAI-compatible service. Here's a vLLM deployment solution, please strictly
   follow our provided startup parameters:

- vLLM:

```shell
python3 -m vllm.entrypoints.openai.api_server \
 --served-model-name autoglm-phone-9b \
 --allowed-local-media-path /   \
 --mm-encoder-tp-mode data \
 --mm_processor_cache_type shm \
 --mm_processor_kwargs "{\"max_pixels\":5000000}" \
 --max-model-len 25480  \
 --chat-template-content-format string \
 --limit-mm-per-prompt "{\"image\":10}" \
 --model zai-org/AutoGLM-Phone-9B \
 --port 8000
```

- This model has the same architecture as `GLM-4.1V-9B-Thinking`. For detailed information about model deployment, you
  can also check [GLM-V](https://github.com/zai-org/GLM-V) for model deployment and usage guides.

- After successful startup, you can access the model service via `http://localhost:8000/v1`. If you deploy the model on
  a remote server, use that server's IP to access the model.

## Using AutoGLM

### Command Line

Set the `--base-url` and `--model` parameters according to your deployed model. For example:

```bash
# Interactive mode
python main.py --base-url http://localhost:8000/v1 --model "autoglm-phone-9b"

# Specify model endpoint
python main.py --base-url http://localhost:8000/v1 "Find the top-rated cinema nearby and navigate me to there by foot"

# List supported apps
python main.py --list-apps
```

### Python API

```python
from phone_agent import PhoneAgent
from phone_agent.model import ModelConfig

# Configure model
model_config = ModelConfig(
    base_url="http://localhost:8000/v1",
    model_name="autoglm-phone-9b",
)

# Create Agent
agent = PhoneAgent(model_config=model_config)

# Execute task
result = agent.run("Open Taobao and search for wireless earphones")
print(result)
```

## Remote Debugging

Phone Agent supports remote ADB debugging via WiFi/network, allowing device control without USB connection.

### Configure Remote Debugging

#### Enable Wireless Debugging on Phone

Make sure the phone and computer are on the same WiFi network, as shown below:

![Enable Wireless Debugging](resources/setting.png)

#### Use Standard ADB Commands on Computer

```bash

# Connect via WiFi, change to the IP address and port shown on your phone
adb connect 192.168.1.100:5555

# Verify connection
adb devices
# Should show: 192.168.1.100:5555    device
```

### Device Management Commands

```bash
# List all connected devices
adb devices

# Connect to remote device
adb connect 192.168.1.100:5555

# Disconnect specific device
adb disconnect 192.168.1.100:5555

# Execute task on specific device
python main.py --device-id 192.168.1.100:5555 --base-url http://localhost:8000/v1 --model "autoglm-phone-9b" "Open Douyin and browse videos"
```

### Python API Remote Connection

```python
from phone_agent.adb import ADBConnection, list_devices

# Create connection manager
conn = ADBConnection()

# Connect to remote device
success, message = conn.connect("192.168.1.100:5555")
print(f"Connection status: {message}")

# List connected devices
devices = list_devices()
for device in devices:
    print(f"{device.device_id} - {device.connection_type.value}")

# Enable TCP/IP on USB device
success, message = conn.enable_tcpip(5555)
ip = conn.get_device_ip()
print(f"Device IP: {ip}")

# Disconnect
conn.disconnect("192.168.1.100:5555")
```

### Remote Connection Troubleshooting

**Connection refused:**

- Ensure device and computer are on the same network
- Check if firewall is blocking port 5555
- Confirm TCP/IP mode is enabled: `adb tcpip 5555`

**Connection dropped:**

- WiFi may have disconnected, use `--connect` to reconnect
- Some devices disable TCP/IP after restart, requiring USB to re-enable

**Multiple devices:**

- Use `--device-id` to specify which device to use
- Or use `--list-devices` to view all connected devices

## Configuration

### Custom SYSTEM PROMPT

Directly modify the configuration file `phone_agent/config/prompts.py`

1. You can inject system prompts to enhance the model's capabilities in specific domains
2. You can inject app names to disable certain apps

### Environment Variables

| Variable                | Description        | Default                    |
|-------------------------|--------------------|----------------------------|
| `PHONE_AGENT_BASE_URL`  | Model API address  | `http://localhost:8000/v1` |
| `PHONE_AGENT_MODEL`     | Model name         | `autoglm-phone-9b`         |
| `PHONE_AGENT_MAX_STEPS` | Max steps per task | `100`                      |
| `PHONE_AGENT_DEVICE_ID` | ADB device ID      | (auto-detect)              |

### Model Configuration

```python
from phone_agent.model import ModelConfig

config = ModelConfig(
    base_url="http://localhost:8000/v1",
    api_key="EMPTY",  # API key (if required)
    model_name="autoglm-phone-9b",  # Model name
    max_tokens=3000,  # Maximum output tokens
    temperature=0.1,  # Sampling temperature
    frequency_penalty=0.2,  # Frequency penalty
)
```

### Agent Configuration

```python
from phone_agent.agent import AgentConfig

config = AgentConfig(
    max_steps=100,  # Max steps per task
    device_id=None,  # ADB device ID (None for auto-detect)
    verbose=True,  # Print debug info (including thinking process and actions)
)
```

### Verbose Mode Output

When `verbose=True`, the Agent outputs detailed information at each step:

```
==================================================
💭 Thinking Process:
--------------------------------------------------
Currently on the system home screen, need to launch the Xiaohongshu app first
--------------------------------------------------
🎯 Executing Action:
{
  "_metadata": "do",
  "action": "Launch",
  "app": "Xiaohongshu"
}
==================================================

... (continue to next step after executing action)

==================================================
💭 Thinking Process:
--------------------------------------------------
Xiaohongshu is open, now need to tap the search box
--------------------------------------------------
🎯 Executing Action:
{
  "_metadata": "do",
  "action": "Tap",
  "element": [500, 100]
}
==================================================

🎉 ================================================
✅ Task Complete: Successfully searched for food guides
==================================================
```

This allows you to clearly see the AI's reasoning process and specific operations at each step.

## Supported Apps

Phone Agent supports 50+ mainstream Chinese apps:

| Category              | Apps                              |
|-----------------------|-----------------------------------|
| Social & Chat         | WeChat, QQ, Weibo                 |
| E-commerce            | Taobao, JD.com, Pinduoduo         |
| Food & Delivery       | Meituan, Ele.me, KFC              |
| Travel                | Ctrip, 12306, Didi                |
| Video & Entertainment | Bilibili, Douyin, iQiyi           |
| Music & Audio         | NetEase Music, QQ Music, Ximalaya |
| Life Services         | Dianping, Amap, Baidu Maps        |
| Content Communities   | Xiaohongshu, Zhihu, Douban        |

Run `python main.py --list-apps` to see the complete list.

## Available Actions

The Agent can perform the following actions:

| Action       | Description                             |
|--------------|-----------------------------------------|
| `Launch`     | Launch an app                           |  
| `Tap`        | Tap at specified coordinates            |
| `Type`       | Input text                              |
| `Swipe`      | Swipe the screen                        |
| `Back`       | Go back to previous page                |
| `Home`       | Return to home screen                   |
| `Long Press` | Long press                              |
| `Double Tap` | Double tap                              |
| `Wait`       | Wait for page to load                   |
| `Take_over`  | Request manual takeover (login/captcha) |

## Custom Callbacks

Handle sensitive operation confirmation and manual takeover:

```python
def my_confirmation(message: str) -> bool:
    """Sensitive operation confirmation callback"""
    return input(f"Confirm execution of {message}? (y/n): ").lower() == "y"


def my_takeover(message: str) -> None:
    """Manual takeover callback"""
    print(f"Please complete manually: {message}")
    input("Press Enter to continue after completion...")


agent = PhoneAgent(
    confirmation_callback=my_confirmation,
    takeover_callback=my_takeover,
)
```

## Examples

Check the `examples/` directory for more usage examples:

- `basic_usage.py` - Basic task execution
- Single-step debugging mode
- Batch task execution
- Custom callbacks

## Development

### Configure Development Environment

Development requires dev dependencies:

```bash
pip install -e ".[dev]"
```

### Run Tests

```bash
pytest tests/
```

### Complete Project Structure

```
phone_agent/
├── __init__.py          # Package exports
├── agent.py             # PhoneAgent main class
├── adb/                 # ADB utilities
│   ├── connection.py    # Remote/local connection management
│   ├── screenshot.py    # Screen capture
│   ├── input.py         # Text input (ADB Keyboard)
│   └── device.py        # Device control (tap, swipe, etc.)
├── actions/             # Action handling
│   └── handler.py       # Action executor
├── config/              # Configuration
│   ├── apps.py          # Supported app mappings
│   └── prompts.py       # System prompts
└── model/               # AI model client
    └── client.py        # OpenAI-compatible client
```

## FAQ

We've listed some common issues and their solutions:

### Device Not Found

Try resolving by restarting the ADB service:

```bash
adb kill-server
adb start-server
adb devices
```

### Text Input Not Working

1. Ensure ADB Keyboard is installed on the device
2. Enable it in Settings > System > Language & Input > Virtual Keyboard
3. The Agent will automatically switch to ADB Keyboard when input is needed

### Screenshot Failed (Black Screen)

This usually means the app is displaying a sensitive page (payment, password, banking apps). The Agent will
automatically detect this and request manual takeover.

### Citation

If you find our work helpful, please cite the following paper:

```bibtex
@article{liu2024autoglm,
  title={Autoglm: Autonomous foundation agents for guis},
  author={Liu, Xiao and Qin, Bo and Liang, Dongzhu and Dong, Guang and Lai, Hanyu and Zhang, Hanchen and Zhao, Hanlin and Iong, Iat Long and Sun, Jiadai and Wang, Jiaqi and others},
  journal={arXiv preprint arXiv:2411.00820},
  year={2024}
}
```