Stage 1: Text Tools Lab

Dates: Oct 30 – Nov 7, 2026 Related RHCSA topic: Lesson 7, Working with Text Files (Sander van Vugt, RHCSA RHEL 10) System: Rocky Linux 10, user student, home folder

Getting the data
bash
cd
curl -O https://www.ndbc.noaa.gov/data/realtime2/46026.txt
ls -lh 46026.txt
-O (capital letter O) saves the download under its original name. Without it, curl prints the data on the screen instead of saving it.
The file was about 600 KB.
Column reference

Each data line has 19 space-separated fields. The two header lines use the same positions.

Field	Name	Meaning	Unit
$1–$5	YY MM DD hh mm	Year, month, day, hour, minute (UTC)	
$6	WDIR	Wind direction	degrees
$7	WSPD	Wind speed	m/s
$8	GST	Gust speed	m/s
$9	WVHT	Wave height	m
$13	PRES	Air pressure	hPa
$14	ATMP	Air temperature	°C
$15	WTMP	Water temperature	°C

Fields 10–12 and 16–19 are other wave, dew point, visibility, pressure-change and tide values.

Tasks
1. Show the start and end of the file
bash
head -n 5 46026.txt     # 2 header lines + 3 data lines
tail -n 3 46026.txt     # last 3 lines

Finding: the newest reading is at the top of the file, the oldest at the bottom (most log files are the other way around). The file covered about 45 days, from Sep 16 to Oct 31.

2. Show only the first data line
bash
head -n 3 46026.txt | tail -n 1

The first data line is line 3, after the two header lines.

3. Show only the two header lines
bash
sed -n 1,2p 46026.txt

1,2p = from line 1 to line 2, print. -n stops sed from printing every other line.

4. Show date, hour and air temperature
bash
cat 46026.txt | awk '{ print $1, $2, $3, $4, $14 }' | head

Finding: the readings come every 10 minutes, so each hour appears six times. Air temperature was missing (MM) in every reading shown.

5. Hide the header (comment) lines
bash
grep -v '^#' 46026.txt | head

^# = lines starting with #; -v inverts the match.

6. Show the readings from one date
bash
grep '^2026 10 31' 46026.txt | head

The quotes keep the pattern together despite its spaces. ^ makes sure the line starts with the date.

7. Count data lines with at least one missing value
bash
grep -v '^#' 46026.txt | grep MM | wc -l

Finding — the header trap: the first header line contains MM too, as the name of the month column. Without removing the header first, the count is one too high:

bash
grep MM 46026.txt | wc -l                    # 1 more than the line above
8. Save date, hour and air temperature to a file, and count
bash
grep -v '^#' 46026.txt | awk '{ print $1, $2, $3, $4, $14 }' > ~/airtemp.txt
wc -l ~/airtemp.txt
grep -v '^#' 46026.txt | wc -l

Both counts matched (6552 on Oct 31), so every data line was kept and no header line got in.

9. Mark missing values as NA (display only)
bash
grep -v '^#' 46026.txt | sed 's/MM/NA/g' | head
s/MM/NA/g: substitute MM with NA, globally (every match on each line).
Removing the header first avoids the same trap as in Task 7: otherwise the month column name would become NA.
sed only changes what is displayed; 46026.txt itself is unchanged.
Missing values are marked as NA, not replaced by 0: a 0 would look like a real reading and distort averages.
Mistakes I made, and what I learned
Mistake	What happened	Lesson
curl without -O	Data printed on screen, no file saved	-O (capital O) saves the file
No file name	head and sed waited for keyboard input	Every command needs input: a file or a pipe. Ctrl+C stops a waiting command.
File name after a pipe	head -n 3 file | tail -n 1 file ignored the pipe	After |, the command reads from the pipe; no file name
awk '{ $1, $14 }'	Syntax error	awk needs print: awk '{ print $1, $14 }'
Counting MM without removing headers	One extra count	Header lines can match a search; filter # lines first
Small typos (airetemp, airtemp,txt, ~/.airetemp)	Wrong or missing files	Read a command once before pressing Enter
