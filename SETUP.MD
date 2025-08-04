# Запуск StereoCrafter на RunPod

Ниже — единый файл с пошаговой инструкцией для настройки окружения, установки зависимостей и запуска инференса StereoCrafter.

---

## 1. Обновление и установка системных пакетов

```bash
apt-get update
apt-get install -y git git-lfs
git lfs install
2. Клонирование репозитория
cd /workspace
git clone --recursive https://github.com/TencentARC/StereoCrafter.git
cd StereoCrafter
3. Установка Python-зависимостей
python3.10 -m pip install --upgrade pip
python3.10 -m pip install -r requirements.txt
4. Сборка Forward-Warp
cd dependency/Forward-Warp
chmod +x install.sh
./install.sh
cd ../../
5. Загрузка весов моделей
cd /workspace/StereoCrafter
mkdir -p weights && cd weights

# Сохраняем креды для HuggingFace
git config --global credential.helper store

# Устанавливаем CLI и логинимся
pip install --upgrade huggingface_hub
huggingface-cli login --token hf_token --add-to-git-credential

# Клонируем модели
git clone https://huggingface.co/stabilityai/stable-video-diffusion-img2vid-xt-1-1
git clone https://huggingface.co/tencent/DepthCrafter
git clone https://huggingface.co/TencentARC/StereoCrafter

cd ..
6. Подготовка папок для видео
mkdir -p source_video outputs
# Скопируйте ваше видео в папку source_video, например:
# cp /path/to/camel.mp4 source_video/camel.mp4
7. Запуск первого этапа (depth splatting)
python depth_splatting_inference.py \
  --pre_trained_path weights/stabilityai/stable-video-diffusion-img2vid-xt-1-1 \
  --unet_path        weights/DepthCrafter \
  --input_video_path source_video/camel.mp4 \
  --output_video_path outputs/camel_splatting_results.mp4
8. Запуск второго этапа (inpainting)
python inpainting_inference.py \
  --pre_trained_path weights/stable-video-diffusion-img2vid-xt-1-1 \
  --unet_path        weights/StereoCrafter \
  --input_video_path outputs/camel_splatting_results.mp4 \
  --save_dir         outputs
