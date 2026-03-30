# Syllaibus: Syllabus for AI

**Syllaibus** is a data format for syllabuses for AI. It is useful to describe scope of knowledge for AI without providing the knowledge itself.

For example if my GitHub repository contains knowledge related to processing text files in Linux, I could provide Syllaibus file (usually named `.syllaibus.yml`) similar to:

```yml
syllabus:
  description: This is syllabus for "Text File Processing in Linux"
  topics:
  - Introduction & Fundamentals
    - What is "text" in Unix philosophy
    - Text vs binary files
    - File encodings (ASCII, UTF-8, UTF-16)
    - Line endings (LF vs CRLF)
  - Basic Text Utilities
    - Display & inspection
        - head
        - tail
        - watch
    - Counting & size analysis
        - wc
        - du
```

## Syllaibus structure

Syllaibuses are usually stored in root of knowledge repository and named `.syllaibus.yml`. Syllaibus file content follows the format below:

```yml
syllabus:
  description:
  topics:
```

Where `description` is description of topic and `topics` contains nested dictionary of topics covered by syllabus (with arbitrary number of nesting).

## Why is it useful?

You can rely on Sylaibus to provide AI the scope of knowledge. Then you can rely on this information when interacting with AI. You can ask about project covered by Syllaibus:

> Have I covered everything from syllabus?

> Test my knowledge about random topic covered by syllabus by asking me question. 

Or about Syllaibus itself:

> Is there anything missing in syllabus?

> Would you add anything else to syllabus?