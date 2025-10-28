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
#!/bin/bash

echo "Extracting module and port details..."

# Extract module name
grep -E "module " design.v | awk '{print "Module Name:", $2}'

# Extract port list
grep -E "input|output" design.v | sed 's/[;,)]//g' | awk '
/input/ {
  dir="Input"
}
/output/ {
  dir="Output"
}
{
  for(i=1;i<=NF;i++) {
    if ($i ~ /\[/) width=$i;
    if ($i ~ /^[A-Za-z_][A-Za-z0-9_]*$/) port=$i;
  }
  if (port != "")
    printf "%-10s %-10s %-10s\n", dir, width, port;
  width=""; port="";
}'
```
Output
```
Extracting module and port details...

Module Name: alu

Direction  Width      Port
Input      [3:0]      A
Input      [3:0]      B
Input      [1:0]      sel
Output     [3:0]      Y
```
