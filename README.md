#  訓練 Bert-Vits 詳細指令

##  準備環境
### 基本環境
- CUDA + PyTorch
- Anaconda Python
- 自建 venv
- 依賴包
    
    `pip install -r requirements`  

###  Git Clone 原始的程式碼
```git clone https://github.com/fishaudio/Bert-VITS2.git```


### 下載 Bert models 到 Bert-Vits
原始碼上有 `bert` 的資料夾，但沒裡面沒有模型檔 (LFS)。先將整理資料夾刪除，再重新下載。

Bert模型地址：

中文：https://huggingface.co/hfl/chinese-roberta-wwm-ext-large

日文：https://huggingface.co/cl-tohoku/bert-base-japanese-v3/tree/main

可以用下面指令直接下載 (要有 git lfs )

```git clone https://huggingface.co/hfl/chinese-roberta-wwm-ext-large```

## 準備訓練資料集
### 素材資料夾格式
建立 `raw` 資料夾，底下依角色分資料夾，分別放入音檔

### 將音檔與逐字稿對映
依照 `filelists/esd.list` 的格式，將角色的逐字稿寫入 `filelists/genshin.list`

### 文字預處理
將中文轉換為發音音素，並切分訓練/測試集，
執行處理檔。

```python preprocess_text.py```

執行完成後檢查 `filelists/genshin_cleaned.list`

### 音檔重採樣
```python resample.py```

執行成功會多一個 `dataset` 資料夾，裡面有重採樣後的音檔。

### 生成 BERT 的音檔訊息
先修改 `bert_gen.py`

Line 24:
```
    if hps.data.add_blank: 
```
改成
```
    if True:
```
Line 46: default=2 可以適度調高，以加快速度。
```
    parser.add_argument("--num_processes", type=int, default=2)
```
執成完後，`dataset/` 下面會出會很多 .pt 檔

## 配置訓練設置
### 下載底模
`logs/[characterName]/`
### configs
`configs/config.json` 中的 `batch_size` 改成 4，在 GTX 3060 上才跑得動
### train_ms.py
```
os.environ['RANK'] = '0'
os.environ['WORLD_SIZE'] = '1'
os.environ['MASTER_ADDR'] = 'localhost'
os.environ['MASTER_PORT'] = '8080'
```

```
python train_ms.py -c configs/config.json -m sunday
```


##  Gastom Version
### 素材資料夾格式
建立 `raw` 資料夾，底下依角色分資料夾，分別放入音檔

### 將音檔與逐字稿對映
將角色的逐字稿寫入 `filelists/genshin.list`

### 文字預處理
```python preprocess_text.py```

執行完成後檢查  `filelists/genshin_cleaned.list`

### 音檔重採樣
```python resample.py```

### 修改底模名字
`logs/pretrained/` 

### 開始訓練
```
python train_ms.py -c configs/config.json -m sunday
```