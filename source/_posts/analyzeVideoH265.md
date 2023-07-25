---

title: 如何解析h265(HEVC)格式视频

date: 2023-07-25 09:25:42

tags: 
- h265 
- HEVC 
---


# 视频播放器

## 1、flv-h265.js(去使用npm)

```tsx
import React, { useRef, useState, useEffect } from 'react';
import type { CSSProperties } from 'react';
import ReactPlayer from 'react-player';
import styles from './index.less';
import Flv from 'flv-h265.js';
import type { ReactPlayerProps } from 'react-player';
//用于进行视频预览
const VideoPreview: React.FC<{
  src: string;
  width?: string | number;
  height?: string | number;
  previewStyle?: CSSProperties;
  playing?: boolean;
}> = (props) => {
  const { src, previewStyle, playing } = props;
  const videoRef = useRef<ReactPlayerProps>();
  const [myVideo, setMyVideo] = useState<HTMLVideoElement>();
  const onReady = () => {
    const divBox: any = videoRef.current?.wrapper;
    const myVideo2: HTMLVideoElement = divBox?.querySelector('video') as HTMLVideoElement;
    setMyVideo(myVideo2);
    console.log(ReactPlayer);
  };
  useEffect(() => {
    if (!myVideo) return;
    const flvPlayer = Flv.createPlayer({
      type: 'mp4',
      url: src,
    });

    flvPlayer.attachMediaElement(myVideo as HTMLVideoElement);
    flvPlayer.load();
    flvPlayer.play();

    return () => {
      flvPlayer.pause();
      flvPlayer.unload();
    };
  }, [myVideo]);
  return (
    <ReactPlayer
      ref={(e) => (videoRef.current = e as ReactPlayerProps)}
      onReady={onReady}
      config={{ file: { attributes: { controlsList: 'nodownload' } } }}
      playing={playing}
      controls
      url={src}
      // width和height不可写入样式中，否则会被默认的“width:680px;height:320px;”覆盖
      width="100%"
      height="100%"
      style={{
        ...previewStyle,
      }}
      className={styles.previewVideo}
    />
  );
};
export default VideoPreview;

```

主要逻辑：flv-h265.js库暴露一个对象，使用该对象中的createPlayer创建一个解码用的实例flvPlayer（要设置解码文件的类型，这里解码的是MP4文件和解码文件的路径，还可以配置其他配置），使用实例上的attachMediaElement方法传入播放器的真实DOM，load（）加载，play（）播放。在页面销毁时，暂停pause，停止加载unload