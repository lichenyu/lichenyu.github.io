title: keras seq2seq 详细注释版
date: 2019/07/05
categories:
- Program Development
tags:
- Tensorflow
---


seq2seq

```python
'''
referrences:
    https://blog.keras.io/a-ten-minute-introduction-to-sequence-to-sequence-learning-in-keras.html
    https://blog.csdn.net/PIPIXIU/article/details/81016974
dataset:
    http://www.manythings.org/anki/
'''


from tensorflow.python.keras.layers import Input, LSTM, Dense
from tensorflow.python.keras.models import Model
import pandas as pd
import numpy as np


# n_input: input one-hot维度；n_output: output one-hot维度
def create_model(n_input, n_output, n_units):
    # ----------训练阶段（decoder有teacher forcing）----------
    # Input实例化Keras张量
    encoder_input = Input(shape=(None, n_input))
    # n_units为LSTM单元中神经元的个数；return_state返回最后时刻的状态h和c
    encoder = LSTM(n_units, return_state=True)
    # call LSTM，获取最后时刻的状态，作为decoder的初始状态
    _, encoder_h, encoder_c = encoder(encoder_input)
    encoder_state = [encoder_h, encoder_c]

    decoder_input = Input(shape=(None, n_output))
    # 训练模型时需要decoder的输出序列来优化，故return_sequences
    decoder = LSTM(n_units, return_sequences=True, return_state=True)
    # call LSTM，获取输出序列。注意，decoder_output shape(?, ?, n_units)
    decoder_output, _, _ = decoder(decoder_input, initial_state=encoder_state)
    # FC + softmax
    decoder_dense = Dense(n_output, activation='softmax')
    # call Dense（FC + softmax），decoder_output shape = (?, ?, n_output)
    decoder_output = decoder_dense(decoder_output)

    # 生成训练模型
    # 第一个参数为训练模型的输入，包含了encoder和decoder的输入，第二个参数为模型的输出，包含了decoder的输出
    model = Model([encoder_input, decoder_input], decoder_output)


    # ----------推理阶段（即预测）----------
    # 调整训练模型（但是weights和biases不变），以适配预测场景
    # 一个直接的需要修改的地方是改变teacher forcing为前一时刻输出
    # encoder其实没有变化，主要修改decoder

    # 直接生成推断encoder模型
    encoder_infer = Model(encoder_input, encoder_state)

    # 准备生产推断decoder模型
    # On the first decoder call, the hidden and cell states from the encoder will be used to initialize the decoder LSTM layer, provided as input to the model directly.
    decoder_state_input_h = Input(shape=(n_units,))
    decoder_state_input_c = Input(shape=(n_units,))
    # initial_state
    decoder_state_input = [decoder_state_input_h, decoder_state_input_c]
    # call LSTM，获取输出
    decoder_infer_output, decoder_infer_state_h, decoder_infer_state_c = decoder(decoder_input, initial_state=decoder_state_input)

    # The decoder must output the hidden and cell states along with the predicted character on each call, so that these states can be assigned to a variable and used on each subsequent recursive call.
    # 当前时刻得到的状态
    decoder_infer_state = [decoder_infer_state_h, decoder_infer_state_c]
    # 当前时刻的输出，call Dense（FC + softmax），decoder_output shape = (?, ?, n_output)
    decoder_infer_output = decoder_dense(decoder_infer_output)

    # 生成推断decoder模型
    # decoder_state_input本身是个list，+运算符append到前面的list
    decoder_infer = Model([decoder_input] + decoder_state_input, [decoder_infer_output] + decoder_infer_state)

    # 可以看出，decoder_infer没法被训练，因为input不易被直接定义
    # 而model就可以很方便的按定义形式给出input、output，进行训练。具体见下文main中使用

    return model, encoder_infer, decoder_infer


# (samples, max_input_seq_len, features)
def predict_chinese(source, encoder_inference, decoder_inference, n_steps, features):
    # encoding
    state = encoder_inference.predict(source)
    # 第一个字符'\t',为起始标志
    predict_seq = np.zeros((1, 1, features))
    predict_seq[0, 0, target_dict['\t']] = 1

    # ----------看一下，这里就是如何使用推理模型了。----------
    # ----------我们的训练模型仅用于获取weights和biases，获取后真正使用的是推理模型了----------
    output = ''
    # 每次用前一次的预测输出（字符），作为输入，来预测下一次的字符，直到预测出了终止符/最大长度
    # n_steps为OUTPUT_SEQ_LENGTH，即最大句子长度
    for i in range(n_steps):
        # 输入前一次的预测输出字符predict_seq，及隐状态h，c
        # 输出yhat因经过FC+softmax，shape = (?, ?, n_output)
        yhat, h, c = decoder_inference.predict([predict_seq] + state)

        # sample=0，time=-1（last），实际上，yhat shape = (1, 1, output_onehot_dim)
        char_index = np.argmax(yhat[0, -1, :])
        char = target_dict_reverse[char_index]
        output += char

        # 本次state为下一次的init state
        state = [h, c]
        # 本次的output为下一次的input
        predict_seq = np.zeros((1, 1, features))
        predict_seq[0, 0, char_index] = 1

        # 遇到终止符则停止
        if char == '\n':
            break
    return output


NUM_SAMPLES = 2000
BATCH_SIZE = 64
EPOCH = 200
N_UNITS = 256


if __name__ == '__main__':

    # ----------处理数据----------
    data_path = 'cmn.txt'

    # 提取NUM_SAMPLES行数据
    # pd.read_table默认分隔符为'\t'（pd.read_csv默认分隔符则为','）
    df = pd.read_table(data_path, header=None).iloc[:NUM_SAMPLES, :]
    df.columns = ['inputs', 'targets']
    # target每句前后增加'\t'、'\n'，作为起始、终止标志
    df['targets'] = df['targets'].apply(lambda x: '\t' + x + '\n')

    # input、target句子列表
    input_texts = df.inputs.values.tolist()
    target_texts = df.targets.values.tolist()
    # input、target字符列表
    input_characters = sorted(list(set(df.inputs.unique().sum())))
    target_characters = sorted(list(set(df.targets.unique().sum())))

    INUPT_SEQ_LENGTH = max([len(i) for i in input_texts])
    OUTPUT_SEQ_LENGTH = max([len(i) for i in target_texts])
    # one-hot --> features，特征维度为one-hot维度，即字符数
    INPUT_FEATURE_LENGTH = len(input_characters)
    OUTPUT_FEATURE_LENGTH = len(target_characters)

    # encoder seq、decoder seq为定长，且decoder每次都输出，故使用max_input_seq_len、max_out_seq_len
    # (samples, max_input_seq_len, features)
    encoder_input = np.zeros((NUM_SAMPLES, INUPT_SEQ_LENGTH, INPUT_FEATURE_LENGTH))
    # (samples, max_out_seq_len, features)
    decoder_input = np.zeros((NUM_SAMPLES, OUTPUT_SEQ_LENGTH, OUTPUT_FEATURE_LENGTH))
    # (samples, max_out_seq_len, features)
    decoder_output = np.zeros((NUM_SAMPLES, OUTPUT_SEQ_LENGTH, OUTPUT_FEATURE_LENGTH))

    # {char: index}
    input_dict = {char: index for index, char in enumerate(input_characters)}
    # {index: char}
    input_dict_reverse = {index: char for index, char in enumerate(input_characters)}
    # {char: index}
    target_dict = {char: index for index, char in enumerate(target_characters)}
    # {index: char}
    target_dict_reverse = {index: char for index, char in enumerate(target_characters)}

    for seq_index, seq in enumerate(input_texts):
        for char_index, char in enumerate(seq):
            # (samples, max_input_seq_len, features)
            encoder_input[seq_index, char_index, input_dict[char]] = 1

    for seq_index, seq in enumerate(target_texts):
        for char_index, char in enumerate(seq):
            decoder_input[seq_index, char_index, target_dict[char]] = 1.0
            # decoder_input = 前一次的decoder_output
            if char_index > 0:
                decoder_output[seq_index, char_index - 1, target_dict[char]] = 1.0


    # ----------训练模型----------
    model_train, encoder_infer, decoder_infer = create_model(INPUT_FEATURE_LENGTH, OUTPUT_FEATURE_LENGTH, N_UNITS)
    model_train.compile(optimizer='rmsprop', loss='categorical_crossentropy')
    model_train.fit([encoder_input, decoder_input], decoder_output, batch_size=BATCH_SIZE, epochs=EPOCH, validation_split=0.2)


    # ----------测试模型----------
    for i in range(100, 200):
        # (samples, max_input_seq_len, features)
        test = encoder_input[i:i + 1, :, :]  # i:i+1保持数组是三维
        out = predict_chinese(test, encoder_infer, decoder_infer, OUTPUT_SEQ_LENGTH, OUTPUT_FEATURE_LENGTH)
        print(input_texts[i])
        print(out)


```