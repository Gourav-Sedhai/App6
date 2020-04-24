# App6
#Motion Detector Final
-------------------------------------------
#Implementing double graph(Not related to motion detector
from bokeh.plotting import figure, output_file, show

p = figure(plot_width=500, plot_height=400, tools = 'pan, reset')

p.title.text = "Earthquakes"
p.title.text_color = "Orange"
p.title.text_font = "times"
p.title.text_font_style = "italic"

p.yaxis.minor_tick_line_color = "Yellow"

p.xaxis.axis_label = "Times"
p.yaxis.axis_label = "Value"

p.line([1,2,3,4,5], [5,6,5,5,3], line_width=2, color="red", alpha=0.5)
p.circle([i*2 for i in [1,2,3,4,5]], [5,6,5,5,3], size=8, color="olive", alpha=0.5)

output_file("Scatter_plotting.html")
show(p)

-----------------------------------------------------

#script1.py
import cv2, time, pandas
from datetime import datetime

first_frame = None
status_list = [None, None]
times = []
df = pandas.DataFrame(columns=["Start", "End"])

video = cv2.VideoCapture(0)

video.read()
time.sleep(2)

while True:
    check, frame = video.read()
    status = 0

    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    gray = cv2.GaussianBlur(gray,(21,21),0)

    if first_frame is None:
        first_frame = gray
        continue

    delta_frame = cv2.absdiff(first_frame,gray)
    thresh_frame = cv2.threshold(delta_frame, 30, 255, cv2.THRESH_BINARY)[1]
    thresh_frame = cv2.dilate(thresh_frame, None, iterations=2)

    (cnts,_) = cv2.findContours(thresh_frame.copy(), cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

    for contour in cnts:
        if cv2.contourArea(contour) < 10000:
            continue
        status = 1

        (x, y, w, h) = cv2.boundingRect(contour)
        cv2.rectangle(frame, (x,y), (x+w, y+h), (0,255,0), 3)
    status_list.append(status)

    status_list = status_list[-2:]

    if status_list[-1] == 1 and status_list[-2] == 0:
        times.append(datetime.now())
    if status_list[-1] == 0 and status_list[-2] == 1:
        times.append(datetime.now())

    cv2.imshow("Gray Frame", gray)
    cv2.imshow("Delta Frame", delta_frame)
    cv2.imshow("Threshold Frame", thresh_frame)
    cv2.imshow("Color Frame", frame)

    key = cv2.waitKey(1)

    if key == ord('q'):
        if status == 1:
            times.append(datetime.now())
        break

print(status_list)
print(times)

for i in range(0, len(times), 2):
    df = df.append({"Start":times[i], "End":times[i+1]}, ignore_index=True)

df.to_csv("Times.csv")

video.release()
cv2.destroyAllWindows 

----------------------------------------------------------------------------------

#plotting.py
from script1 import df
from bokeh.plotting import figure, show, output_file
from bokeh.models import HoverTool, ColumnDataSource

df["Start_string"] = df["Start"].dt.strftime("%Y-%m-%d %H:%M:%S")
df["End_string"] = df["End"].dt.strftime("%Y-%m-%d %H:%M:%S")

cds = ColumnDataSource(df)

p = figure(x_axis_type='datetime', height=100, width=500, sizing_mode='scale_width', title="Motion Graph")
p.yaxis.minor_tick_line_color = None
p.yaxis[0].ticker.desired_num_ticks = 1

hover = HoverTool(tooltips=[("Start","@Start_string"), ("End","@End_string")])
p.add_tools(hover)

q = p.quad(left="Start", right="End", bottom=0, top=1, color="green", source=cds)

output_file("Graph.html")

show(p)

--------------------------------------------------
The End of Application 6
--------------------------------------------------
