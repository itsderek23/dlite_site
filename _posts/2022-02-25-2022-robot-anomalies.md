---
layout: post
title:  "Effortlessly Investigating Robot Anomalies at Greenzie"
date: 2022-02-25 05:00:00 -0600
canonical_url: https://www.greenzie.com/post/effortlessly-investigating-robot-anomalies-at-greenzie
---

_Originally published on the Greenzie blog, this covers a project I led to make it easier to debug problems on a fleet of autonomous commercial lawnmowers._

The life of a [Greenzie-equipped autonomous mower](http://greenzie.com/product) is 97% boring punctuated by small anomalies. This presents two debugging challenges for our ROS developers:

1.  There's little need for rich data the vast majority of time, but when there is an anomaly, there's a thirst for ALL the data.
2.  Our mobile fleet of robotic workers are customer-operated (not managed by robotics technicians), do not dock to a fast Internet connection, and are frequently turned on and off. Getting this data from a robot to a developer's laptop is a challenge.

Easy access to debugging data is important to us as we're big believers in [Kaizen](https://en.wikipedia.org/wiki/Kaizen). It should be easy for developers to continually improve our software, and a big part of that is making it effortless to view debugging data. We've recently rolled out an update to our developer tooling that makes obtaining and viewing high-fidelity ROS data silky-smooth. Let's see how it works.

End-user experience overview
----------------------------

![](/img/posts/greenzie_rosbag/foxglove_screens.png)

The end user of our anomaly data fetch system is either a ROS developer or a support engineer. Here's how the system is typically used:

1.  A support engineer reviews a robot job and notices an anomaly. For example: the robot generated an obstacle alert. Robots post alerts to our platform and these are rendered as markers on a satellite map. The user clicks the alert marker for details and the "Get ROS Bag" button.
2.  A new [Foxglove](https://foxglove.dev) marker is added to the map along with the path of the robot over the duration of the ROS bag file.
3.  The ROS bag data is uploaded to the cloud via rsync, then uploaded to [Foxglove's data platform](https://foxglove.dev/data-platform). The end user is notified via email when the upload is complete. The user can click on a link from the marker popup to view the data on Foxglove.
4.  The user views the data with [Foxglove Studio](https://foxglove.dev/studio). Foxglove Studio can be used in the browser or via their desktop apps. The ROS stack does need to be installed on the user's computer.

‍

Technical Details
-----------------

![](/img/posts/greenzie_rosbag/foxglove_diagram.png)

Delivering the end user experience above involves a couple pieces of custom work and one of our favorite new robotics developer tools, [Foxglove](https://foxglove.dev). Here's how it works:

1.  Our custom admin web app running in the cloud creates a database record with the robot identifier, timestamp, and duration of the data fetch.
2.  The app checks every minute if the robot is online. If so, any unsent requests for ROS data are sent to the robot.
3.  The robot marks ROS bag files that occurred during the requested timeframe for transfer to the cloud. Bag files are split in 30 second increments. An rsync transfer is started via Cron to send the requested ROS bags to the cloud. Cron is used so that partially transferred files will resume upload automatically when the robot is power cycled.
4.  Back in the cloud, the app monitors for completed rsync file transfers. Once completed, files are uploaded to the Foxglove Data Platform via their API. The user that requested the ROS bag data is notified via email of the completed data request. They can then immediately view the data within Foxglove Studio.

Some notes on the technical details:

1.  **Keep on-robot logic simple** - we try to limit the complexity of utilities that run on our robots in the field. The greatest complexity of this flow resides in the cloud. In the cloud, we can deploy fixes in minutes and easily debug issues (always-on fast internet, always-on servers, no disk space concerns, and a mature ecosystem of monitoring and debugging tools).
2.  **Limit resource usage on the robot** - there are few cases where the ROS data is needed to debug an immediate problem. Instead, we use the data from these anomalies to prevent future problems. We limit the resource usage of the on-robot parts of this flow in several ways: flock to prevent a thundering heard of rsync processes, limits on rsync transfer size, deriving the ROS bags to collect based on the last modified time of the file and not loading and inspecting files, etc.
3.  [**Foxglove**](http://foxglove.dev/) - this SaaS fulfills two important needs: (1) storage of ROS bag data (2) effortless viewing of ROS bag data (no ROS stack required). I'm very excited about their vision to continue improving both of these areas.

Potential for automated anomaly data collection
-----------------------------------------------

A developer must manually trigger the ROS data fetch: we'll always need this ability. However, I'm excited that this flow for fetching data can work in an automated fashion too: these are just basic HTTP APIs. For example:

1.  **Anomaly detection in the cloud** - we send a limited amount of data to the cloud every minute. This could be analyzed with an anomaly detection algorithm, automatically triggering a data fetch on an anomaly event.
2.  **Anomaly detection on the robot** - it's also easy for a ROS Node to trigger the API if it notices an anomaly. For example:  trigger a data fetch if a computer vision ML Model appears to be confused.

TL;DR
-----

We're excited to have a lean, "right-sized" stack for easily investigating incidents from the field on our robots. Web developers have leveraged these kinds of easy-to-use tools for years (ex: Sentry, Scout, DataDog, etc). It's great to see more of these as mobile robots move beyond prototypes to the real world.
