---
layout: post
title: useROS 커스텀 훅으로 React에서 ROSlibjs 사용하기
date: 2023-11-08 17:40:20 +0900
tags: [Tech, Development, Robotics]
usemathjax: false
---

<fig >
<img src="https://images.velog.io/images/qaszx1004/post/378c7512-f24e-4fb5-965d-6bf377acb608/ROS.jpg">
</fig>

최근 회사에서 ROS 시스템에 들어가는 Dashboard를 React를 이용해 만드는 일을 하고있다. 웹에서 ROS에 접근할 수 있게 해주는 ROSLIB 패키지를 사용하고 있는데, 대충 구조는 이렇다.

<fig>
<img src="https://www.researchgate.net/publication/271510079/figure/fig8/AS:614393071017997@1523494131460/Dataaow-between-the-Browser-and-the-Web-Server-The-Browser-requests-the-webpage-from.png">
<figcaption>Profanter, Stefan. (2014). Implementation and Evaluation of multimodal input/output channels for task-based industrial robot programming. </figcaption>
</fig>

해당 라이브러리에서 제공하는 rosbridge package를 설치하면, 사용자의 ros환경에 rosbridge_server라는 node를 생성할 수 있다. 그러면 해당 node는 웹의 websocket과 HTTP 통신을 열어 topic, service, action과 관련된 정보를 JSON 형태로 주고받을 수 있게 된다. Web side에서는 roslibjs 라이브러를 사용하여 이 JSON형태의 데이터를 받아올 수 있다.

공식 Docs에서 제공하는 코드는 바닐라js환경을 기반으로 하고 있어서, React 프레임워크에서 이용한 예시를 찾기 힘들었다. 그래서 이번에 만들어야 할 React기반 웹페이지에서 roslibjs를 범용적으로 사용될 수 있는 useROS라는 커스텀 훅을 사용했다.

```ts
// useROS.ts
import { useEffect, useState } from "react";
import ROSLIB from "roslib";

const useROS = () => {
  const rosObj = {
    ROS: new ROSLIB.Ros({}),
    url: "ws://localhost:9090",
    isConnected: false,
    ROSConfirmedConnected: false,
    autoconnect: true,
    topics: [],
    services: [],
    listeners: [],
  };

  const [ros, setROS] = useState<typeof rosObj>(rosObj);

  useEffect(() => {
    if (!ros.isConnected) {
      if (ros.autoconnect) {
        console.log("autoconnecting");
        handleConnect();
      }
    }
    checkConnection();
  }, []);

  const toggleConnection = () => {
    if (ros.isConnected) {
      handleDisconnect();
    } else if (!ros.isConnected) {
      handleConnect();
    }
  };

  const toggleAutoconnect = () => {
    if (ros.autoconnect) {
      setROS((ros) => ({ ...ros, autoconnect: false }));
    } else if (!ros.autoconnect) {
      setROS((ros) => ({ ...ros, autoconnect: true }));
    }
  };

  const changeUrl = (new_url: string) => {
    setROS((ros) => ({ ...ros, url: new_url }));
  };

  const checkConnection = () => {
    if (ros.ROS) {
      if (ros.isConnected) {
        if (!ros.ROSConfirmedConnected && ros.ROS.isConnected) {
          setROS((ros) => ({
            ...ros,
            ROSConfirmedConnected: ros.ROS.isConnected,
          }));
          console.log("Both react-ros and roslibjs have confirmed connection.");
        }
        // Once we have that "confirmation"  we need to continously check for good connection
        else if (ros.ROSConfirmedConnected && !ros.ROS.isConnected) {
          setROS((ros) => ({ ...ros, isConnected: false }));
          handleDisconnect();
        } else if (!ros.ROS.isConnected) {
          console.log(
            "React-ros has confirmed the connection, roslibjs has not yet."
          );
        }
      }
    } else {
      console.log("Initial connection not established yet");
    }
  };

  const handleConnect = () => {
    try {
      ros.ROS.connect(ros.url);
      ros.ROS.on("connection", () => {
        setROS((ros) => ({ ...ros, isConnected: true })); // seems to take awhile for the roslibjs library to report connected
        setROS((ros) => ({ ...ros, ROSConfirmedConnected: false }));
        getTopics();
      });

      ros.ROS.on("error", (error) => {
        //gets a little annoying on the console, but probably ok for now
        console.log(error);
      });
    } catch (e) {
      console.log(e);
    }
  };

  const handleDisconnect = () => {
    try {
      ros.ROS.close();
      setROS((ros) => ({ ...ros, isConnected: false }));
      setROS((ros) => ({ ...ros, topics: [] }));
      setROS((ros) => ({ ...ros, listeners: [] }));
      setROS((ros) => ({ ...ros, ROSConfirmedConnected: false }));
    } catch (e) {
      console.log(e);
    }
    console.log("Disconnected");
  };

  const getTopics = () => {
    const topicsPromise = new Promise((resolve, reject) => {
      ros.ROS.getTopics(
        (topics) => {
          const topicList = topics.topics.map((topicName, i) => {
            return {
              path: topicName,
              msgType: topics.types[i],
              type: "topic",
            };
          });
          resolve({
            topics: topicList,
          });
          reject({
            topics: [],
          });
        },
        (message) => {
          console.error("Failed to get topic", message);
          ros.topics = [];
        }
      );
    });
    topicsPromise.then((topics: any) => {
      setROS((ros) => ({ ...ros, topics: topics.topics }));
    });
    return ros.topics;
  };

  /**
   * @param topic name of ros2 topic to subscribe
   * @param messageType type of ros2 topic to subscribe
   * @param callback
   */
  const subscribeToTopic = (
    topic: string,
    messageType: string,
    callback: (message: any) => void
  ) => {
    const rosTopic = new ROSLIB.Topic({
      ros: ros.ROS,
      name: topic,
      messageType,
    });

    rosTopic.subscribe(callback);

    return () => {
      rosTopic.unsubscribe();
    };
  };

  /**
   * ROS2 topic publisher
   * @param topic name of ros2 topic to subscribe
   * @param messageType type of ros2 topic to subscribe
   * @param message message to publish. must be in messageType
   */
  const publishToTopic = (topic: string, messageType: string, message: any) => {
    const publisher = new ROSLIB.Topic({
      ros: ros.ROS,
      name: topic,
      messageType,
    });
    console.log(`===publish to Topic function : ${topic}===`);
    publisher.publish(message);
    return () => {
      console.log(`===return observed to ${topic}===`);
      publisher.unsubscribe();
    };
  };

  return {
    checkConnection,
    toggleConnection,
    changeUrl,
    toggleAutoconnect,
    publishToTopic,
    subscribeToTopic,
    handleConnect,
    ros: ros.ROS,
    isConnected: ros.isConnected,
    autoconnect: ros.autoconnect,
    url: ros.url,
    topics: ros.topics,
    services: ros.services,
    listeners: ros.listeners,
  };
};

export default useROS;
```
