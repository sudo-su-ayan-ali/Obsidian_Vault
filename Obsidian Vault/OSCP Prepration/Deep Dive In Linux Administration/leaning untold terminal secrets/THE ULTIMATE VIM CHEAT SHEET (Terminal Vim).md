k# 🧠🔥 THE ULTIMATE VIM CHEAT SHEET (Terminal Vim)

---

## 🔁 MODES (THIS IS EVERYTHING)

```
Normal     Esc        move, delete, change
Insert     i a o      type text
Visual     v V Ctrl+v select text
Command    :          commands
Replace    R          overwrite text
```

### Enter Insert mode

```
i   insert before cursor
a   insert after cursor
I   insert at start of line
A   insert at end of line
o   new line below
O   new line above
```

---

## 🚶 MOVEMENT (NORMAL MODE)

### Basic

```
h  left
j  down
k  up
l  right
```

### Words

```
w  next word start
e  end of word
b  previous word

W E B   BIG words (ignore punctuation)
```

### Line

```
0   start of line
^   first non-space
$   end of line
```

### Screen / File

```
Ctrl+d   half page down
Ctrl+u   half page up
Ctrl+f   full page down
Ctrl+b   full page up

gg       top of file
G        bottom of file
:n       go to line n
```

---

## ✏️ INSERT MODE POWER MOVES

```
Ctrl+o + motion   run ONE normal command
Ctrl+w            delete word backward
Ctrl+u            delete to start of line
Ctrl+h            backspace
Ctrl+j            newline
```

---

## 🎯 VISUAL MODE (SELECTION)

```
v        select characters
V        select lines
Ctrl+v   block (column) select
```

### Smart selections

```
viw   select word
vaw   select word + space
v$    select to end of line
v0    select to start of line
```

### Actions on selection

```
d  delete
y  yank (copy)
c  change
>  indent right
<  indent left
```

---

## ❌ DELETE

```
x     delete char
X     delete char backward
dw    delete word
db    delete backward word
diw   delete whole word
dd    delete line
D     delete to end of line
```

---

## 🔄 CHANGE (DELETE + INSERT)

```
cw    change word
ciw   change whole word
cc    change line
C     change to end of line
r     replace one character
R     replace mode
```

---

## 📋 COPY / PASTE (YANK)

```
yy    yank line
yw    yank word
y$    yank to end of line
p     paste after cursor
P     paste before cursor
```

Registers:

```
"ay   yank into register a
"ap   paste from register a
```

---

## ↩️ UNDO / REDO

```
u       undo
U       undo whole line
Ctrl+r  redo
```

---

## 🔍 SEARCH & NAVIGATION

```
/word   search forward
?word   search backward
n       next match
N       previous match

*       search word under cursor
#       search backward word
```

---

## 🔁 FIND IN LINE

```
fX   find X forward
FX   find X backward
tX   move before X
TX   move after X
;    repeat
,    reverse repeat
```

---

## 🔢 COUNTS (MULTIPLY EVERYTHING)

```
5j     move down 5 lines
3dw    delete 3 words
10x    delete 10 chars
```

---

## 🧠 TEXT OBJECTS (VIM MAGIC)

```
iw   inner word
aw   a word
ip   inner paragraph
ap   a paragraph
i"   inside quotes
a"   around quotes
i(   inside parentheses
```

Examples:

```
ciw   change word
di"   delete inside quotes
va(   select around ()
```

---

## 🧱 BLOCK / MULTI-LINE EDITING

```
Ctrl+v → select block
I      insert before all lines
A      append after all lines
d      delete block
```

---

## 🔁 REPEAT & MACROS

```
.      repeat last action

qa     start macro (register a)
q      stop macro
@a     run macro
@@     run last macro
```

---

## 🪄 INDENTATION

```
>>   indent line
<<   unindent line
=    auto-indent selection
```

---

## 🧾 FILES & BUFFERS

```
:e file      open file
:w           save
:q           quit
:wq          save & quit
:q!          quit without saving
```

Buffers:

```
:ls
:b next
:b prev
:bd   delete buffer
```

---

## 🪟 WINDOWS & TABS

### Splits

```
:sp     horizontal split
:vsp    vertical split
Ctrl+w h/j/k/l   move between splits
Ctrl+w q         close split
```

### Tabs

```
:tabnew
gt       next tab
gT       previous tab
```

---

## ⚙️ SETTINGS (COMMON)

```
:set number
:set relativenumber
:set nowrap
:set ignorecase
:set smartcase
```

---

## 🧠 GOLDEN RULE OF VIM

```
[number] + verb + motion
```

Examples:

```
3dw   delete 3 words
ci"   change inside quotes
y$    copy to end of line
```

---

## 🏆 IF YOU MASTER JUST THESE, YOU’RE SOLID

```
h j k l
w b
0 $
gg G
v V Ctrl+v
dw diw ciw
u Ctrl+r
/pattern
.
```

---

If you want next:

- a **printable one-page PDF**
    
- a **beginner vimrc**
    
- or a **“modern editor” Vim setup**
    

Say the word 😎