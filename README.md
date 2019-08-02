# auto-speedtest-log
Run an automated internet speedtest, log the data and visualize it every 24 hours

The following allows you to test your internet speed in set intervals, write the data in a log file and create visualizations (plots) of the data once a day.

Follow these steps to set you up:

## 0) Running linux enginge with python (should be included in most distros) and established internet connection
I am running it on a RaspberryPi using Raspbian 10 Buster. Check what OS you are running with 
```
cat /etc/os-release
```
Creating a working directory
```
mkdir /path/to/speedtest
```
Change to that directory
```
cd /path/to/speedtest
```
For the rest of this readme every command is given presuming you are in this directory.

## 1) Implementing speedtest.py
This project uses speedtest.py from https://github.com/sivel/speedtest-cli . You can get it using wget and make it executable:
```
wget https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py
sudo chmod +x speedtest.py
```
Testing it with 
```
./speedtest.py
```
should prompt an output like
```
Retrieving speedtest.net configuration...
Testing from [provider] (your.IP)...
Retrieving speedtest.net server list...
Selecting best server based on ping...
Hosted by [server hosting] (location) [distance km]: 26.317 ms
Testing download speed................................................................................
Download: 9.79 Mbit/s
Testing upload speed................................................................................................
Upload: 5.43 Mbit/s
```
## 2) Prepare logfile
```
./speedtest.py --csv-header > speedtest.log
```
will create the logfile `speedtest.log` and write the csv headers in the first line:
```
cat speedtest.log
Server ID,Sponsor,Server Name,Timestamp,Distance,Ping,Download,Upload,Share,IP Address
```
Do a test of the speedtest log with: _(this takes a moment)_
```
./speedtest.py --csv >> speedtest.log
```
then cat the logfile should prompt:
```
cat speedtest.log
Server ID,Sponsor,Server Name,Timestamp,Distance,Ping,Download,Upload,Share,IP Address
10010,[sponsor.url],[location],2019-08-02T19:51:52.311122Z,13.089912652947309,28.738,12271838.6438877,5826553.658862884,,IP.##.##.##
```
That is the beginning of you logfile. __Note: the timestamp in the logfiles is in [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time)!__

If you want to empty it again simply overwrite it with the csv headers: `./speedtest.py --csv-header > speedtest.log`. _(note: `>` overwrites the file, `>>` appends a line to the file, be careful)_
## 3) Install gnuplot and the gnuplot script
On debian based distributions (like Raspbian) install gnuplot using:
```
sudo apt-get install gnuplot
```
Get the gnuplot script i prepared 
```
wget https://raw.githubusercontent.com/MOTSM-HD/auto-speedtest-log/master/gnuplot_speedtest
```
and customize it according to your needs, especially the working directory. See comments in script.
## 4) set up scheduled tasks using cron
Always edit the crontab using `crontab -e`. This way when you save the changes it will be effective immediatly, no need to restart cron. Each cronjob is a line at the end of the crontab. _The crontab may be empty when you begin!_

We need two cronjobs:
```
*/30 * * * * /path/to/speedtest/speedtest.py --csv >> /path/to/speedtest/speedtest.log
5 2 * * * gnuplot /path/to/speedtest/gnuplot_speedtest
```
The first will run the speedtest python script and append its csv output to the logfile two times per hour. more specifically: *every hour at 00 and 30 minutes*.

The second will run gnuplot using the given script once a day, in this case every night at 02:05 am. The results will be one plot for the last 24 hours as .png file per day.

