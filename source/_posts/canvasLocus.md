---

title: 通过canvas画轨迹

date: 2023-09-13 17:30:02

tags: 
- canvas
- 画笔 
---


# 画笔
{% dplayer "formula" url="/assets/video/canvas.mp4"  custom="{width: '100px', height: '100px'}" %}
```tsx
import React, { useRef } from 'react';

const dotwidth = 5600;
// const dotheight = 7560;
const dotheight = 7920;
function App() {
  const penRef = useRef<TinsoPen.TinsoPenRef>({
    isConnected: false,
    isPenDown: false,
    strokeData: {},
    currentStrokeData: [],
    flag: false,
    currentPt:{i:0,k:0}
  });
  // const flag = {bool:true};
 
  
  const data: Record<string, [x: number, y: number][][]> = {
    "1536.736.22.24": [[[1338, 695], [1336, 692], [1337, 690], [1341, 690], [1370, 689], [1432, 683], [1545, 671], [1697, 671], [1876, 673], [2067, 685], [2268, 707], [2450, 737], [2598, 768], [2717, 794], [2808, 812], [2863, 824], [2889, 830], [2888, 828]],
      [[1281, 749], [1277, 746], [1281, 743], [1308, 746], [1369, 747], [1476, 751], [1625, 744], [1808, 742], [2014, 749], [2216, 763], [2402, 779], [2557, 803], [2693, 824], [2802, 845], [2881, 860], [2923, 871], [2926, 873]],
      [[1601, 963], [1580, 953], [1538, 945], [1523, 949], [1551, 951], [1623, 950], [1727, 948], [1860, 946], [2018, 951], [2190, 961], [2342, 980], [2463, 998], [2550, 1010], [2599, 1012], [2617, 1015], [2603, 1011]],
      [[1583, 1129], [1542, 1127], [1482, 1119], [1471, 1122], [1489, 1118], [1573, 1110], [1706, 1089], [1871, 1071], [2062, 1066], [2251, 1071], [2418, 1078], [2534, 1086], [2601, 1097], [2629, 1101], [2626, 1098]]],
    "1536.736.22.14": [[[1427, 1477], [1425, 1477], [1424, 1478], [1426, 1478], [1427, 1478]],
      [[398, 954], [400, 954], [404, 958], [408, 964], [410, 974], [414, 991], [414, 1018], [411, 1050], [414, 1071], [413, 1092], [413, 1107], [409, 1114], [405, 1123]],
      [[499, 978], [498, 977], [500, 978], [503, 977], [511, 980], [526, 989], [537, 996], [544, 1005], [542, 1012], [524, 1025], [500, 1043], [480, 1061], [468, 1071], [469, 1072], [474, 1072], [489, 1070], [508, 1069], [526, 1066], [540, 1062], [546, 1060], [555, 1056]],
      [[636, 939], [638, 938], [645, 938], [668, 942], [690, 949], [698, 956], [694, 964], [680, 977], [656, 992], [647, 1002], [649, 1008], [664, 1008], [688, 1011], [700, 1015], [702, 1022], [695, 1030], [680, 1045], [661, 1062], [651, 1075], [648, 1081]],
      [[795, 934], [795, 933], [796, 932], [794, 935], [785, 950], [766, 979], [754, 1012], [754, 1020], [766, 1023], [780, 1018], [800, 1015], [821, 1014], [849, 1011], [873, 1009], [881, 1006], [884, 1004]],
      [[830, 925], [824, 919], [823, 912], [822, 912], [822, 913], [826, 921], [830, 940], [836, 971], [840, 1011], [846, 1046], [852, 1097], [857, 1143], [861, 1188], [863, 1209], [865, 1215], [864, 1214], [863, 1212], [862, 1211]]]
  }

 /**根据点阵位置计算真实canvas坐标
   * @param canvas 网页的canvas元素
   * @param coor 点阵坐标
   */

 const calculateRealCoor = (canvas: HTMLCanvasElement, coor: [x: number, y: number]) => {
  return {
    x: (canvas?.clientWidth / dotwidth) * coor[0],
    y: (canvas?.clientHeight / dotheight) * coor[1],
  };
};

  const drawStrokeDynamic = (
    pt: { x: number; y: number },
    tension: number,
    color = 'rgb(199,116,99)',
  ) => {
    const canvas = document.querySelector('#aiPen') as HTMLCanvasElement;
    const ctx = canvas?.getContext('2d');
    if (ctx) {
      if (!penRef.current.lastPt) {
        penRef.current.lastPt = pt;
      } else {
        ctx.beginPath();
        ctx.moveTo(penRef.current.lastPt.x, penRef.current.lastPt.y);

        ctx.lineJoin = 'round';
        ctx.lineWidth = 1;
        ctx.strokeStyle = color; // "#000"
        ctx.fillStyle = 'rgb(0,0,255)';
        const controlPt: { x: number; y: number } = { x: 0, y: 0 };
        const control_scale = (tension / 0.5) * 0.175;
        controlPt.x = penRef.current.lastPt.x + (pt.x - penRef.current.lastPt.x) * control_scale;
        controlPt.y = penRef.current.lastPt.y + (pt.y - penRef.current.lastPt.y) * control_scale;
        ctx.quadraticCurveTo(controlPt.x, controlPt.y, pt.x, pt.y);
        penRef.current.lastPt = pt;
        ctx.stroke();

        // ctx.stroke();
        ctx.closePath();
      }
    }
  };

 const drawCurrentPage = (
    pageAddress: string,  // 变量对象的属性，例："1536.736.22.24"
    color = 'rgb(199,116,99)',  //轨迹的颜色
    lineData?: Record<string, [x: number, y: number][][]>,  //data数据
    isDynamic?: boolean,  //时候动态作画
    dynamicOptions?: TinsoPen.DynamicProps,
  ) => {
    const canvas = document.querySelector('#aiPen') as HTMLCanvasElement;
  //清理定时器，防止有定时器未被清理
    if (penRef.current.timer) {
      clearTimeout(penRef.current.timer);
      penRef.current.timer = undefined;
      penRef.current.lastPt = undefined;
    }
   //改动态画笔参数，不用管
    const { speedInNum, defaultSpeed } = dynamicOptions || {}; //此处接收动态画笔的参数
    let realSpeed = 30;
    if (speedInNum) {
      realSpeed = speedInNum;
    } else if (defaultSpeed) {
      switch (defaultSpeed) {
        case 'fast':
          realSpeed = 20;
          break;
        case 'normal':
          realSpeed = 30;
          break;
        case 'slow':
          realSpeed = 40;
          break;
        default:
          break;
      }
    }

    /** 动态画上笔记数据（展示画笔过程）
     * @param data 某一页的笔记数据
     * @param i 默认为0，某次抬笔落笔数据的索引
     * @param k 默认为0，某次抬笔落笔数据索引的笔坐标信息的索引
     * @param res Promise的resolve函数，可以作笔记数据画完后的回调函数
     * @param rej Promise的reject函数，可以作data参数为空时的失败回调函数
     * */
   
    const drawProgressDynamic =async (
      data: [x: number, y: number][][],
      i = 0,
      k = 0,
      res: (value: boolean) => void,
      rej: (value: boolean) => void,
    ) => {
     
      if (data) {
        penRef.current.timer =  setTimeout(() => {
          const coInfo = data[i][k];
          drawStrokeDynamic(calculateRealCoor(canvas, coInfo), 1, color); //画轨迹

          /*
          drawStrokeDynamic({x轴位置,y轴位置}, 1, color);
          */
          //前两个判断表示如果没跑到数组的最后一组参数，就继续递归当前这个函数（满足前两个条件就一直跑）
          //第三个判断表示，跑到最后一组参数了，重置位置的索引，方便点击后重新运行。异步函数结束调用成功回调
          //最后的判断，是递归被暂停的情况（画到一半停止）。点击改变penRef.current.flag=false,也是调用异步成功的回调（我主动暂停的怎么不算成功呢）。保
          //保存当前的位置索引
          if ((k < data[i].length - 1)&&penRef.current.flag) {
            drawProgressDynamic(data, i, k + 1, res, rej);
          } else if ((i < data.length - 1 )&&penRef.current.flag) {
            penRef.current.lastPt = undefined;
            drawProgressDynamic(data, i + 1, 0, res, rej);
          }
          else if (k == data[i].length - 1 && i == data.length - 1) {
            penRef.current.currentPt = ({ i: 0, k: 0 });
            penRef.current.lastPt = undefined;
            clearTimeout(penRef.current.timer);
            penRef.current.timer = undefined;
            res(true);
            }
          else {
            penRef.current.lastPt = undefined;
            clearTimeout(penRef.current.timer);
            penRef.current.timer = undefined;
            penRef.current.currentPt = ({ i, k });
            res(true);
          }
        }, realSpeed);
      } else {
        rej(false);
      }
    };
      //此处正式开始跑的逻辑。如果数据data存在，跑第一个判断，然后如果动态展示isDynamic为true，跑第一个的第一个判断
    if (lineData) {
      if (isDynamic) {
        return new Promise<boolean>((res, rej) => {
          drawProgressDynamic(lineData?.[pageAddress], penRef.current?.currentPt.i,penRef.current?.currentPt.k, res, rej);
        }
          
        );
      } else {
        lineData?.[pageAddress]?.forEach((r) => {
          r.forEach((k) => drawStrokeDynamic(calculateRealCoor(canvas, k), 1, color));
          penRef.current.lastPt = undefined;
        });
      }
    } else {
      penRef.current.strokeData?.[pageAddress]?.forEach((r) => {
        r.forEach((k) => drawStrokeDynamic(calculateRealCoor(canvas, k), 1));
        penRef.current.lastPt = undefined;
      });
    }
    return Promise.resolve(true);
  };
  
 
  return (<div >
   <canvas style={{border:'1px solid black'}} id='aiPen' width={600} height={1000}></canvas>
    <button onClick={() => {
      penRef.current.flag = !penRef.current.flag;
      //轨迹跑完了会将位置的索引重置为0,又因为此时代码需要在运行，所以penRef.current.flag=true。清空画布
      if (penRef.current.currentPt.i===0&& penRef.current.currentPt.k===0&&penRef.current.flag) {
        const canva = document.querySelector('#aiPen') as HTMLCanvasElement;
        const ctxCanvas = canva.getContext('2d');
        ctxCanvas?.clearRect(0,0,canva.width,canva.height);
      }
      //penRef.current.flag=true时，需要运行画轨迹的函数。让后这个函数正常结束的话，必定是被暂停的状态（不过是手动暂停，还是跑完了自动暂停的）
      if (penRef.current.flag) {
        drawCurrentPage("1536.736.22.24", 'rgb(199,116,99)', data, true).then(()=>penRef.current.flag=false
        );
      }
    
      
    }} style={{ position: 'fixed', top: '50%', left: '50%' }}>开始/暂停</button>
    
    </div>);
}

export default App;

```
通过一个异步递归函数，来运行笔记的轨迹。通过按钮的点击来打断函数的运行，在通过一个变量currentPt来存储被打断的地方的位置的索引（用来定位的数组的下标），再次点击从被打断位置继续之前的函数运行。如果函数正常结束，则再次点击时，清空页面内容，重置下标索引为初始位置。
