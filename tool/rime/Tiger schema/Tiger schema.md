```yaml
# Rime schema settings
# encoding: utf-8

schema:
  schema_id: xt
  name: "我的老虎"
  version: "0.1"
  author:
    - 發明人
  description: |
    虎码
  dependencies:
    - pinyin_simp

switches:
  - name: ascii_mode
    reset: 0
    states: [ 中文, 西文 ]
  - name: full_shape
    states: [ 半角, 全角 ]
  - name: extended_charset
    states: [ 常用, 增廣 ]
  - name: ascii_punct
    states: [ 。，, ．， ]

engine:
  processors:
    - ascii_composer
    - recognizer
    - key_binder
    - speller
    - punctuator
    - selector
    - navigator
    - express_editor
  segmentors:
    - ascii_segmentor
    - matcher
    - abc_segmentor
    - punct_segmentor
    - fallback_segmentor
  translators:
    - punct_translator
    - reverse_lookup_translator
    - table_translator

speller:
  delimiter: " ;'"
  #max_code_length: 4  # 四碼頂字上屏

translator:
  dictionary: tigress
  prism: xt
  enable_charset_filter: true
  enable_sentence: true
  enable_encoder: true
  encode_commit_history: true
  max_phrase_length: 4

abc_segmentor:
  extra_tags:
    - reverse_lookup

reverse_lookup:
  dictionary: pinyin_simp
  prefix: "`"
  suffix: "'"
  tips: 〔拼音〕
  preedit_format:
    - xform/([nl])v/$1ü/
    - xform/([nl])ue/$1üe/
    - xform/([jqxy])v/$1u/

punctuator:
  import_preset: default

key_binder:
  import_preset: default

recognizer:
  import_preset: default
  patterns:
    reverse_lookup: "`[a-z]*'?$"

```

方案依赖官方的词库。