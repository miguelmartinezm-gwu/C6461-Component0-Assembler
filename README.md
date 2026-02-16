# C6461 Assembler - Component 0

CS6461 Computer Architecture Project - Component 0

## Team Members
Team 7:
Chandle Mubashir,
Mukendi Patrick Tshiswaka, 
Miguel Martinez

## Project Description

This is a two-pass assembler for the C6461 Computer Architecture. It translates C6461 assembly language source code into machine code.

## How to Build and Run

### In IntelliJ IDEA:
1. Open project in IntelliJ
2. Run → Run 'C6461Assembler'
3. Output files will be in `test/` folder

### From Command Line:
```bash
# Compile
javac -d out/production/C6461_Assembler src/edu/gwu/cs6461/assembler/*.java

# Run
java -cp out/production/C6461_Assembler edu.gwu.cs6461.assembler.C6461Assembler test/test_program.asm
```

### Using JAR File:
```bash
java -jar out/artifacts/C6461_Assembler_jar/C6461_Assembler.jar test/test_program.asm
```

## Output Files

- `.lst` - Listing file with addresses and machine code
- `.load` - Load file for simulator (octal format)

## Project Structure
```
C6461_Assembler/
├── src/edu/gwu/cs6461/assembler/
│   ├── C6461Assembler.java    - Main assembler (two-pass algorithm)
│   ├── Instruction.java        - Instruction encoding
│   ├── InstructionSet.java     - All C6461 instructions
│   └── SourceLine.java         - Source line parsing
└── test/
    └── test_program.asm        - Test assembly program
```

## Features

- Two-pass assembly algorithm
- Support for all C6461 instructions
- Label support with forward references
- Error detection and reporting
-  Octal output format

## Deliverables for Component 0

- Java source code
-  JAR file (in out/artifacts/)
- Test program
- Documentation
