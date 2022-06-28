# Convert md to docx

- t.sh
```sh
# pandoc에서 '  - '로 스페이스 2칸이면 들여쓰기가 무시되므로 '    - '와 같이 스페이스 4칸으로 변경
cat "${1}" | sed 's/^\( *\)\*/\1\1-/g' | sed 's/^\( *\)-/\1\1-/g' | sed 's/^\( *\)\([0-9]\.\)/\1\1\2/g' > tmp.md
mv tmp.md "${1}"

# find 실행 결과 ‘./t.md’와 같이 나오면 pandoc에서 에러나므로 ‘t.md’와 같이 변경
file=$(echo "${1}" | sed 's/^\.\///g')
pandoc "${file}" -o "${file}.docx"

# md 파일 삭제 및 docx 파일 rename 
rm "${1}"
rename 's/.md.docx/.docx/g' "${1}.docx"
```

- ta.sh
```sh
find . -maxdepth 1 -name '*.md' -exec ./t.sh {} \;
```

# Typora

## Install

```sh
wget -qO - https://typora.io/linux/public-key.asc | sudo apt-key add -
sudo add-apt-repository 'deb https://typora.io/linux ./'
sudo apt-get update
sudo apt-get install typora
```

## Config

- 파일 > 환경설정
  - 모양 > 상태바, 왼쪽 패널
  - 이미지 > Copy image to custom folder, ./images/${filename}, 가능하다면 상대적 위치 사용



## Change Width of Writing Area

파일 > 환경설정 > 모양 > 테마 폴더 열기 선택

base.user.css 추가

```css
#write {
  max-width: 1300px;
}
```



## Draw Diagrams

> https://support.typora.io/Draw-Diagrams-With-Markdown/

### Sequence Diagrams

> https://bramp.github.io/js-sequence-diagrams/#syntax

```sequence
Alice->Bob: Hello Bob, how are you?
Note right of Bob: Bob thinks
Bob-->Alice: I am good thanks!
```

### Flowcharts

> http://flowchart.js.org/

```flow
st=>start: Start
op=>operation: Your Operation
cond=>condition: Yes or No?
e=>end

st->op->cond
cond(yes)->e
cond(no)->op
```

### Mermaid

>  https://mermaid-js.github.io/mermaid/#/

#### Sequence Diagrams

> https://mermaid-js.github.io/mermaid/#/sequenceDiagram

```mermaid
%% Example of sequence diagram
sequenceDiagram
Alice->>Bob: Hello Bob, how are you?
alt is sick
Bob->>Alice: Not so good :(
else is well
Bob->>Alice: Feeling fresh like a daisy
end
opt Extra response
Bob->>Alice: Thanks for asking
end
```

#### Flowcharts

>  https://mermaid-js.github.io/mermaid/#/flowchart

```mermaid
graph LR
A[Hard edge] -->B(Round edge)
B --> C{Decision}
C -->|One| D[Result one]
C -->|Two| E[Result two]
```

#### Gantt Charts

>  https://mermaid-js.github.io/mermaid/#gantt

```mermaid
%% Example with selection of syntaxes
gantt
dateFormat  YYYY-MM-DD
title Adding GANTT diagram functionality to mermaid

section A section
Completed task            :done,    des1, 2014-01-06,2014-01-08
Active task               :active,  des2, 2014-01-09, 3d
Future task               :         des3, after des2, 5d
Future task2               :         des4, after des3, 5d

section Critical tasks
Completed task in the critical line :crit, done, 2014-01-06,24h
Implement parser and jison          :crit, done, after des1, 2d
Create tests for parser             :crit, active, 3d
Future task in critical line        :crit, 5d
Create tests for renderer           :2d
Add to mermaid                      :1d

section Documentation
Describe gantt syntax               :active, a1, after des1, 3d
Add gantt diagram to demo page      :after a1  , 20h
Add another diagram to demo page    :doc1, after a1  , 48h

section Last section
Describe gantt syntax               :after doc1, 3d
Add gantt diagram to demo page      : 20h
Add another diagram to demo page    : 48h
```

#### Class Diagrams

>  https://mermaid-js.github.io/mermaid/#/classDiagram

```mermaid
classDiagram
Animal <|-- Duck
Animal <|-- Fish
Animal <|-- Zebra
Animal: +int age
Animal: +String gender
Animal: +isMammal()
Animal: +mate()
class Duck{
  +String beakColor
  +swim()
  +quack()
}
class Fish{
  -int sizeInFeet
  -canEat()
}
class Zebra{
  +bool is_wild
  +run()
}
```

#### State Diagrams

>  https://mermaidjs.github.io/#/stateDiagram

```mermaid
stateDiagram
[*] --> Still
Still --> [*]

Still --> Moving
Moving --> Still
Moving --> Crash
Crash --> [*]
```

#### Pie Charts

```mermaid
pie
title Pie Chart
"Dogs" : 386
"Cats" : 85
"Rats" : 150
```

# References
> [Typora](https://typora.io/#windows) 
>
> > [Typora: Change Width of Writing Area](https://support.typora.io/Width-of-Writing-Area/) 
> >
> > [Typora: Draw Diagrams With Markdown](https://support.typora.io/Draw-Diagrams-With-Markdown/) 
> >
> > [Mermaid Diagram Syntax](https://mermaid-js.github.io/mermaid/#/) 
>
> [Milkdown](https://milkdown.dev/) 
>
> > [Milkdown: GitHub](https://github.com/Saul-Mirone/milkdown) 
> >
> > [Milkdown: VSCode extension](https://marketplace.visualstudio.com/items?itemName=mirone.milkdown) 

