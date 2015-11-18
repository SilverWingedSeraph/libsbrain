# libsbrain
A library for execution of Semantic Brain, based on Urban Müller's famous but unprintable language.

### What is SBrain?
  SBrain, or Semantic Brain, is a language based on Urban Müller's famous language with only 8 symbols (3 bit instructions). SBrain's additions increase the number of symbols to 32 (6 bit instructions) and adds a stack and a register.

### Specification
#### Data Structures
  SBrain requires:
  
* a read/write tape datastructure which is addressable up to, at minimum, 65,535 (0x0 - 0xFFFF) 32-bit cells. Not all of these must be active in memory; however, SBrain programs may assume that they are all addressable. They must be initially set to zero unless set with an initialization instruction.
* a read/write stack (FILO) datastructure which must support, at minimum, 256 values. Not all of these must be active in memory; however, SBrain programs may assume that they are addressable. They must be initially set to zero.
* a read-only tape datastructure which contains the executable code. This code is represented as a list of unsigned integers of, at minimum, six bits in width.
* a read-only nonreversable tape containing the program's input (note: as this tape is nonreversable and nonwriteable, a function like C's getch() works fine.)
* a write-only nonreversable tape containing the program's output (note: as this tape is nonreversable and nonwriteable, a function like C's putch() works fine.)
* a read/write register (`data_p`) of enough bits to store a position on the data tape
* a read/write register (`inst_p`) of enough bits to store a position on the instruction tape
* a read/write register (`jump_p`) of enough bits to store a position on the instruction tape
* a read/write register (`auxi_r`) of the same size as a cell on the data tape

#### Commands and Source Code

SBrain source code consists of text characters. Executable code consists of unsigned integers of six bits. A transliterator converts the source code to executable code by a one-to-one mapping, with two exceptions. The first is noted in the entry for instruction 31 (@), which is a metacharacter in certain circumstances. The second is the comment character, #. All data between # characters, including those characters, is ignored by the transliterator.

Decimal | Code  | Semantics
--------|-------|----------
       0|      <|Decrement `data_p`
       1|      >|Increment `data_p`
       2|      -|Subtract one from the cell pointed at by `data_p`
       3|      +|Add one to the cell pointed at by `data_p`
       5|      [|Set `jump_p` to the current position and, if the cell pointed at by `data_p` is zero, cease evaluating instructions until `inst_p` points at either a 6 (`]`), in which case begin evaluating there, or a 
       6|      ]|Set `inst_p` to `jump_p` if the cell pointed at by `data_p` is nonzero.
       7|      .|Place the value in the cell pointed at by `data_p` on the output tape
       8|      ,|Place the next value from the input tape in the cell pointed at by `data_p`
       9|      {|Push the value from the cell pointed at by `data_p` onto the stack
      10|      }|Pop the next value from the stack into the cell pointed at by `data_p`
      11|      (|Set `auxi_r` to the value of the cell pointed at by `data_p`
      12|      )|Set the cell pointed at by `data_p` to the value in `auxi_r`
      13|      z|Set the value in `auxi_r` to 0
      14|      !|Perform a bitwise NOT on the value in `auxi_r`.
      15|      s|Perform a bitshift to the left on the value in `auxi_r`. Bits shifted off the left are lost, and bits shifted in from the right are always zero.
      31|      @|End the program. The exit code is the value in `auxi_r`. If repeated twice (@@) in the source code, the transliterator will consider all further source code to be data and will use it to initialize the data tape.

The following instructions are seperated because they all follow similar rules. Each one performs an operation on the value at the cell pointed to by `data_p` (`a`) and the value in `auxi_r`(`b`), in that order if the operation is not commutative, storing it in the cell pointed at by `data_p`. The creation of a value in a cell greater than the maximum value able to be held by that cell shall result in a wraparound (e.g. 0xFFFFFFFF + 0b11 = 0b11)

Decimal | Code  | Semantics
--------|-------|----------
      16|     \|| a OR b (bitwise)
      17|      &| a AND b (bitwise)
      18|      *| a XOR b (bitwise)
      19|      ^| a NOR b (bitwise)
      20|      $| a NAND b (bitwise)
      21|      s| SUM of a and b
      22|      d| DIFFERENCE of a and b
      23|      p| QUOTIENT of a and b (a divided by b)
      24|      m| a MODULO b
      25|      p| PRODUCT of a and b (a multiplied by b)


#### Further Rules
No read operation shall ever disrupt a cell on the data tape.
