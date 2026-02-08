# C6461 Assembler - Design Notes
## Component 0 - CS6461 Computer Architecture

Team Members: Chandle Mubashir, Tshiswaka Mukendi, Miguel Martinez

Date: 02/07/2026
Version: 1.0

---

## 1. Design Overview

The C6461 Assembler implements a classic two-pass assembly algorithm to translate C6461 assembly language into machine code.

### Design Goals
- Translate all C6461 instructions correctly
- Support labels and forward references
- Generate proper octal output format
- Detect and report errors clearly
- Maintainable, object-oriented code structure

---

## 2. Architecture

### Class Structure

The assembler consists of 4 main classes:

1. C6461Assembler - Main controller, orchestrates two-pass algorithm
2. SourceLine - Represents and parses one line of source code
3. Instruction - Handles instruction encoding for all formats
4. InstructionSet - Static repository of all C6461 instructions

### Class Responsibilities

#### C6461Assembler.java (Main Class)

Purpose: Orchestrates the assembly process

Key Methods:
- assemble() - Main entry point
- pass1() - Build symbol table
- pass2() - Generate machine code
- readSourceFile() - Parse input
- writeListingFile() - Generate .lst file
- writeLoadFile() - Generate .load file

Data Structures:
- Symbol table: HashMap for O(1) label lookup
- Source lines: ArrayList for sequential processing
- Error list: ArrayList for error collection

#### SourceLine.java

Purpose: Represents one parsed source line

Responsibilities:
- Parse line into components (label, opcode, operands, comment)
- Store assembly address and encoded value
- Provide getters for all components

Key Features:
- Handles labels (ending with colon)
- Separates comments (starting with semicolon)
- Splits operands by comma

#### Instruction.java

Purpose: Represents an instruction type with encoding logic

Format Types:
- LOAD_STORE - Memory operations
- REGISTER - Register-to-register operations
- IMMEDIATE - Immediate value operations
- SHIFT_ROTATE - Bit manipulation
- IO - Input/output operations
- MISC - Miscellaneous (HLT, TRAP)

Encoding Methods:
- Each method builds 16-bit instruction using bit shifts and OR
- Returns proper format for each instruction type

#### InstructionSet.java

Purpose: Central repository of all C6461 instructions

Implementation:
- Static HashMap initialized at class load
- Maps mnemonic to Instruction object
- Contains all 40+ C6461 instructions

Categories:
- Load/Store (LDR, STR, LDA, LDX, STX)
- Arithmetic (AMR, SMR, AIR, SIR, MLT, DVD)
- Logical (AND, ORR, NOT, TRR)
- Branch (JZ, JNE, JCC, JMA, JSR, RFS, SOB, JGE)
- Shift/Rotate (SRC, RRC)
- I/O (IN, OUT, CHK)
- Floating Point (FADD, FSUB, VADD, VSUB, CNVRT, LDFR, STFR)

---

## 3. Two-Pass Algorithm

### Pass 1: Symbol Table Construction

Purpose: Build symbol table and assign addresses

Algorithm:
```
locationCounter = 0
FOR EACH source line:
    Parse line into components
    
    IF line has label:
        IF label exists in symbolTable:
            ERROR: duplicate label
        ELSE:
            symbolTable[label] = locationCounter
    
    IF opcode is "LOC":
        locationCounter = operand value
    ELSE IF opcode is "DATA" or valid instruction:
        line.address = locationCounter
        locationCounter = locationCounter + 1
```

Error Detection:
- Duplicate labels
- Invalid opcodes
- Invalid LOC values

### Pass 2: Code Generation

Purpose: Generate machine code and resolve labels

Algorithm:
```
locationCounter = 0
FOR EACH source line:
    IF opcode is "LOC":
        locationCounter = operand value
    
    ELSE IF opcode is "DATA":
        IF operand is label:
            value = symbolTable[operand]
        ELSE:
            value = parse operand as integer
        line.encodedValue = value
    
    ELSE IF valid instruction:
        Encode instruction based on format
        Resolve any label references
        line.encodedValue = encoded instruction
```

Label Resolution:
- Look up label in symbol table
- Substitute with address value
- Error if label not found

---

## 4. Instruction Encoding

### Bit Field Layout

All instructions are 16 bits. Format varies by type.

Load/Store Format:
```
Bits: 15-10 | 9-8 | 7-6 | 5 | 4-0
      OpCode|  R  | IX  | I | Address
```

Example: LDR 3,0,10
```
OpCode: LDR = 0x01 = 000001
R: 3 = 11
IX: 0 = 00
I: 0 (not indirect)
Address: 10 (decimal) = 01010

Encoding: 000001 11 00 0 01010
Binary: 0000011100001010
Octal: 003412
```

Register Format:
```
Bits: 15-10 | 9-8 | 7-6 | 5-0
      OpCode| Rx  | Ry  | Unused
```

Immediate Format:
```
Bits: 15-10 | 9-8 | 7-5    | 4-0
      OpCode|  R  | Unused | Immediate
```

### Encoding Process

1. Get opcode and format from InstructionSet
2. Parse operands based on format
3. Resolve label references if needed
4. Combine fields using bitwise operations
5. Return 16-bit result

---

## 5. File Formats

### Input: Assembly Source (.asm)

Format:
```
[label:] [opcode] [operand1,operand2,...] [;comment]
```

Example:
```
 LOC 10
Start: LDR 0,0,20    ;Load register 0
 AIR 0,5         ;Add 5
 JZ 0,0,End      ;Jump if zero
End: HLT             ;Stop
```

### Output: Listing File (.lst)

Format:
```
<6-digit octal address> <6-digit octal value> <original source>
```

Example:
```
000012 003420 Start: LDR 0,0,20    ;Load register 0
000013 014005  AIR 0,5         ;Add 5
```

### Output: Load File (.load)

Format:
```
<6-digit octal address> <6-digit octal value>
```

Example:
```
000012 003420
000013 014005
```

---

## 6. Error Handling

### Error Categories

1. Syntax Errors
    - Invalid opcode
    - Missing operands
    - Malformed operands

2. Semantic Errors
    - Duplicate label
    - Undefined label reference
    - Invalid register number
    - Address out of range

3. I/O Errors
    - File not found
    - Permission denied

### Error Reporting

Format: ERROR: Line <number>: <message>

Example:
```
ERROR: Line 5: Duplicate label 'Start'
ERROR: Line 12: Unknown instruction 'LOAD'
```

---

## 7. Design Decisions

### Why Two-Pass?

Decision: Use two-pass approach

Rationale:
- Handles forward references naturally
- Clear separation of concerns
- Standard approach for assemblers
- Easier to debug and test

Alternative Considered: One-pass with backpatching
- More complex to implement
- Harder to maintain
- Not needed for our scale

### Why HashMap for Symbol Table?

Decision: HashMap<String, Integer>

Rationale:
- O(1) average lookup time
- No size limitations
- Standard Java collection
- Simple to use

Alternative Considered: Array-based table
- Would need to predict size
- Linear search O(n)
- More complex implementation

### Why Object-Oriented Design?

Decision: Separate classes for each responsibility

Rationale:
- Modularity - each class has one job
- Testability - can test independently
- Maintainability - easy to modify
- Extensibility - easy to add features

Benefits:
- SourceLine handles parsing
- Instruction handles encoding
- InstructionSet handles lookup
- C6461Assembler orchestrates everything

---

## 8. Testing Strategy

### Unit Testing Approach

Each class tested independently:
- SourceLine: Parse various line formats
- Instruction: Encode each format type
- InstructionSet: Lookup all instructions
- C6461Assembler: Full assembly process

### Integration Testing

- Complete programs from documentation
- Edge cases (labels, forward refs, errors)
- All instruction types
- Both directives (LOC, DATA)

### Test Coverage

Instruction Types:
- Load/Store (LDR, STR, LDA, LDX, STX)
- Arithmetic (AIR, SIR, AMR, SMR)
- Logical (AND, ORR, NOT)
- Branch (JZ, JNE, JMA)
- Miscellaneous (HLT)

Features:
- Labels and forward references
- Directives (LOC, DATA)
- Comments
- Error detection

---

## 9. Code Quality

### Coding Standards

- Naming: CamelCase for classes, camelCase for methods/variables
- Comments: Javadoc for all public methods
- Indentation: 4 spaces
- Line length: < 100 characters
- Error handling: Try-catch for I/O operations

### Documentation

- Inline comments for complex logic
- Javadoc comments for public API
- README for usage instructions
- This design document for architecture

---

## 10. Performance

### Time Complexity

- Pass 1: O(n) where n = number of lines
- Pass 2: O(n) where n = number of lines
- Symbol lookup: O(1) average (HashMap)
- Overall: O(n) linear time

### Space Complexity

- Symbol table: O(m) where m = number of labels
- Source lines: O(n) where n = number of lines
- Overall: O(n + m) linear space

### Practical Performance

- Small programs (< 100 lines): < 0.1 seconds
- Medium programs (< 1000 lines): < 1 second
- Large programs (< 10000 lines): < 5 seconds

---

## 11. Known Limitations

1. Number Format: Only decimal input (no hex/octal in source)
2. Expressions: No arithmetic in operands
3. Macros: Not supported
4. Multi-file: Single source file only

These are intentional simplifications for Component 0.

---

## 12. Future Enhancements

Possible additions for future components:
- Macro definitions and expansion
- Expression evaluation in operands
- Include file support
- Conditional assembly
- Optimization passes
- Better error messages with suggestions

---

## 13. Lessons Learned

1. Two-pass simplifies design - Worth the extra pass
2. HashMap ideal for symbol table - Fast and simple
3. OOP provides clarity - Each class has clear purpose
4. Error handling important - Good error messages save debugging time
5. Testing incrementally - Test each class as built

---

Project Status: Complete and tested
Last Updated: 02/07/2026
Contributors: Team D