---

title: 选定答题位置

date: 2023-09-21 09:51:42

tags: 
- 事件监听 
- DOM操作 
---
{% dplayer "formula" url="/assets/video/getPosition.mp4"  custom="{width: '100px', height: '100px'}" %}
```tsx
import React, { useEffect, useRef, useState } from 'react';
import { Document, Page } from 'react-pdf';  //渲染pdf到页面
import { PageContainer } from '@ant-design/pro-components';
import { Spin } from 'antd';
import styles from './index.less';

const AnswerArea = () => {
  const [pageNum, setPageNum] = useState<number>(0);  //储存pdf的页数
  const startPtRef = useRef<{ x: number; y: number }>({ x: 0, y: 0 });  //记录开始位置
  const endPtRef = useRef<{ x: number; y: number }>({ x: 0, y: 0 });  //记录结束位置
  const actionRef = useRef<boolean>(false);  //判断时候是操作状态
  const boxRef = useRef<HTMLDivElement>();  //记录当前的元素，用于辅助页面渲染

  const [allPt, setAllPt] = useState<[[number, number], [number, number]][]>([]);  //存储所有位置信息用于渲染
  //拿到数据，这边换成接口拿，接口还未写好，暂时用本地存储代替
  useEffect(() => {
    setAllPt(JSON.parse(localStorage.getItem('positions') || '[]'));
  }, []);  
  // 判断作答区域是否有重叠的现象
  const isOverlap = (data: [[number, number], [number, number]][]) => {
    return data.every((item) => {
      const x1 = item[0][0];
      const y1 = item[0][1];
      const x2 = item[1][0];
      const y2 = item[1][1];
      const curX1 = startPtRef.current.x;
      const curY1 = startPtRef.current.y;
      const curX2 = endPtRef.current.x;
      const curY2 = endPtRef.current.y;
      return curX2 < x1 || x2 < curX1 || curY2 < y1 || y2 < curY1;
    });
  };

  return (
    <PageContainer title="操作">
      <Spin spinning={!pageNum}>
        <div
          onMouseDown={(e) => {
            actionRef.current = true;  //开始，用useRef可以实时保存，每次拿到的都是最新的值
            const page = document.querySelector('#page') as HTMLDivElement;
            const rect = page.getBoundingClientRect();
            const left = e.clientX - rect.left;
            const top = e.clientY - rect.top;  //获取位置（相对于#page的位置，因为用的绝对定位）
            startPtRef.current = { x: left, y: top };  //存储开始位置
            const div = document.createElement('div');   //创建div用于页面的渲染
            const style = `width:1px;height:1px;background:rgba(3,3,3,0.1);position:absolute;top:${top}px;left:${left}px;`;
            div.setAttribute('style', style);
            boxRef.current = div;  //存下来用于后，后面的删除
            page.appendChild(div);
          }}
          onMouseMove={(e) => {
            if (actionRef.current) {
              const page = document.querySelector('#page') as HTMLDivElement;
              const rect = page.getBoundingClientRect();
              const currentDiv = boxRef.current as HTMLDivElement;
              currentDiv.style.width = e.clientX - rect.left - startPtRef.current.x + 'px';
              currentDiv.style.height = e.clientY - rect.top - startPtRef.current.y + 'px';  //鼠标移动时，实时改变元素的宽高
            }
          }}
          onMouseUp={(e) => {
            actionRef.current = false;  //一次选区结束
            const page = document.querySelector('#page') as HTMLDivElement;
            const rect = page.getBoundingClientRect();
            endPtRef.current = { x: e.clientX - rect.left, y: e.clientY - rect.top };  //存储结束位置
            if (isOverlap(allPt)) {
              //如果选的区域重叠了，这边不运行，数据不能添加到allPt中，也不会传到后端（暂时用本地存储实现）
              const start = startPtRef.current;
              const end = endPtRef.current;
              //这个判断是防止用户点一下，也产生一个数据
              if (end.x - start.x > 10 && end.y - start.y > 10) {
                allPt.push([
                  [start.x, start.y],
                  [end.x, end.y],
                ]);
                setAllPt([...allPt]);
                localStorage.setItem('positions', JSON.stringify(allPt));
              }
            }
            boxRef.current?.remove();  //把页面通过js创建的选区用的div删除，因为上面数据已经添加到allPt变量中，页面会实时更新
          }}
          onMouseLeave={(e) => {
            //这边是离开操作区域的逻辑，以离开前的位置渲染
            const page = document.querySelector('#page') as HTMLDivElement;  
            const rect = page.getBoundingClientRect();
            endPtRef.current = { x: e.clientX - rect.left, y: e.clientY - rect.top };
            if (isOverlap(allPt)) {
              const start = startPtRef.current;
              const end = endPtRef.current;
              if (end.x - start.x > 10 && end.y - start.y > 10 && actionRef.current) {
                actionRef.current = false;
                if (page.offsetWidth < end.x) {
                  //从右边离开
                  allPt.push([
                    [start.x, start.y],
                    [page.offsetWidth - 4, end.y],
                  ]);
                } else if (page.offsetHeight < end.y) {
                  //从下面离开
                  allPt.push([
                    [start.x, start.y],
                    [end.x, page.offsetHeight - 7],
                  ]);
                } else {
                  //这边写上更保险点
                  allPt.push([
                    [start.x, start.y],
                    [end.x - 3, end.y - 2],
                  ]);
                }

                setAllPt([...allPt]);
                localStorage.setItem('positions', JSON.stringify(allPt));
              }
            }
            boxRef.current?.remove();
          }}
          id="page"
          className={styles.content}
        >
          <Document
            file={'http://127.0.0.1:5501/testPaper.pdf'}
            onLoadSuccess={({ numPages }) => setPageNum(numPages)}
          >
            {Array.from({ length: pageNum }, (_, index) => index + 1).map((item) => (
              <div className={styles.itemPage} key={`page${item}`}>
                <Page
                  renderTextLayer={false}
                  renderAnnotationLayer={false}
                  pageNumber={item}
                  width={592.28 * 1.4}
                  height={841.89 * 1.4}
                />
              </div>
            ))}
          </Document>

          {allPt.map((item, i) => (
            <div
              className={styles.boxContent}
              key={`${item[0][0]}&${item[0][1]}`}
              style={{
                width: `${item[1][0] - item[0][0]}px`,
                height: `${item[1][1] - item[0][1]}px`,
                top: `${item[0][1]}px`,
                left: `${item[0][0]}px`,
              }}
            >
              <div
                onMouseDown={(e) => e.stopPropagation()}
                onMouseMove={(e) => e.stopPropagation()}
                onMouseUp={(e) => e.stopPropagation()}
                data-position={`${`del${item[0][0]}&${item[0][1]}`}`}
                className={`${styles.del} del`}
                onClick={() => {
                  const newAllPt = allPt.filter((_, index) => index !== i);
                  setAllPt(newAllPt);
                  localStorage.setItem('positions', JSON.stringify(newAllPt));
                }}
              >
                删除
              </div>
            </div>
          ))}
        </div>
      </Spin>
    </PageContainer>
  );
};

export default AnswerArea;


```
通过监听事件，mousedown，mousemove，mouseup，mouseleave操作，有选区重叠的判断，和离开操作区域的判断。
