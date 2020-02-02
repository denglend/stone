 
 
 1. Converting weather icons
 2. Getting data from Wunderground

Weather icons in SVG format from: https://github.com/erikflowers/weather-icons/.  

Converting SVG to use in Arduino
 - Inkscape, export as PNG (height = 100)
 - GIMP to reformat and export as PBM 
	 - Layers -> Transparency -> Remove Alpha Channel
	 - Colors -> Threshold
	 - File -> Export As -> .pbm
 - Python script for .pbm to .h

```python
import sys
if len(sys.argv) != 3:
	print('{:s} infile outfile'.format(sys.argv[0]))
	sys.exit()
fout = open(sys.argv[2],"w")
with open(sys.argv[1],"rb") as fin:
	print(fin.readline())
	print(fin.readline())
	sizeline = fin.readline()
	x,y = sizeline.split(" ")
	print(x)
	print(y)
	fout.write(x+","+y)
	b = fin.read(1)
	c = 0
	while b:
		fout.write('0x{:02X}'.format(ord(b)))
		b = fin.read(1)
		if b: fout.write(", ")
		c += 1
		if c%20 == 0: fout.write("\n")

fout.close()
```
