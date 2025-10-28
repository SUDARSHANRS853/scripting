## To automatically extract module names, input/output ports, and port widths from Verilog RTL files using Linux text-processing tools (awk, grep, sed).
Tool Used
```
 grep → to locate module definitions
 sed → to remove unwanted characters (like commas, semicolons)
 awk → to extract and format port information
```
Design.v
```
module alu (input [3:0] A, input [3:0] B, input [1:0] sel, output reg [3:0] Y);
always @(*) begin
  case(sel)
    2'b00: Y = A + B;
    2'b01: Y = A - B;
    2'b10: Y = A & B;
    2'b11: Y = A | B;
  endcase
end
endmodule
```
Script(port_extraction.sh)
```
#!/usr/bin/bash                       # Use bash shell to execute the script

echo "Extracting module and port details..."  # Display starting message
echo                                          # Print a blank line

#  MODULE NAME EXTRACTION 
grep -E "module " design1.v | awk '{print "Module Name:", $2}'  
# grep → search for line containing "module"
# awk → print "Module Name:" followed by 2nd field (module name)

echo                                          # Print blank line for spacing

#  PRINT TABLE HEADER 
printf "%-10s %-10s %-10s\n" "Direction" "Width" "Port"  
# Print column headings (Direction, Width, Port) with fixed spacing
echo "-------------------------------"         
# Print separator line

#  PORT EXTRACTION 
grep -E "input|output" design1.v | sed 's/[;,)]//g' | awk '  
# grep → find lines containing "input" or "output"
# sed  → remove characters ; , and ) for cleaner parsing
# awk  → begin processing each line

/input/  { dir="Input"  }                     # If line has "input", set direction as Input
/output/ { dir="Output" }                     # If line has "output", set direction as Output

{
  for(i=1;i<=NF;i++) {                        # Loop through each word (field) in the line
    if ($i ~ /\[/) width=$i;                  # If field has [ ] → it's the width (e.g. [3:0])
    if ($i ~ /^[A-Za-z_][A-Za-z0-9_]*$/) port=$i;  # If field matches Verilog identifier pattern → port name
  }
  if (port != "")                             # If port name found
    printf "%-10s %-10s %-10s\n", dir, width, port; # Print Direction, Width, and Port columns
  width=""; port="";                          # Reset width and port for next line
}'

```
Output
```
Extracting module and port details...

Module Name: alu

Direction  Width      Port
-------------------------------
Input      [3:0]      A
Input      [3:0]      B
Input      [1:0]      sel
Output     [3:0]      Y
```


```
1.Indicates script should run using Bash.
2.Displays message about starting extraction process.
3.Finds lines with module in design.v and prints the module name.
4.This command finds all lines in design.v containing input or output, removes commas, semicolons, and parentheses using sed, then uses awk to check each line — if it has input, it sets dir="Input", and if it has output, it sets dir="Output" — preparing for further port extraction or formatting.
5.This AWK block loops through every word (i=1 to NF) in each line — if a word contains [ it saves it as width (like [7:0]), and if a word matches a valid Verilog identifier pattern (letters, digits, or underscore starting with a letter/underscore), it saves it as the port name.
6.If a port name was found (port != ""), it prints the direction, width, and port name in neatly aligned columns using printf, then resets width and port to empty for processing the next line.
