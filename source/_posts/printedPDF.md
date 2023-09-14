---

title: 前端页面转PDF（适配不会有元素被分割）

date: 2023-09-4 17:30:20

tags:
- toPDF
- html2canvas
- JsPDF
---

# 打印文件

## 1、html2canvas&&JsPDF
{% dplayer "formula" url="/assets/video/toPDF.mp4"  custom="{width: '100px', height: '100px'}" %}
```
$ yarn add html2canvas
$ yarn add jspdf
```

```
import html2canvas from 'html2canvas';
import JsPDF from 'jspdf';
```

```tsx
const getPdf = (title: string, dom: string) => {
    return new Promise((resolve, reject) => {
      html2canvas(document.querySelector(dom) as HTMLDivElement, {
        useCORS: true,
        allowTaint: true,
        ignoreElements: (ele) => {
          if (ele.className === 'blanks') {
            return true;
          }
          return false;
        },
      })
        .then(function (canvas) {
          const contentWidth = canvas.width;
          const contentHeight = canvas.height;
          // 根据A4纸的大小，计算出dom相应比例的尺寸
          const pageHeight = (contentWidth / 592.28) * 841.89;
          let leftHeight = contentHeight;
          let position = 0;
          const imgWidth = 595.28;
          // 根据a4比例计算出需要分割的实际dom位置
          const imgHeight = (592.28 / contentWidth) * contentHeight;
          // canvas绘图生成image数据，1.0是质量参数
          const pageData = canvas.toDataURL('image/jpeg', 1.0);
          // a4大小
          const PDF = new JsPDF('p', 'pt', 'a4');
          if (leftHeight < pageHeight) {
            PDF.addImage(pageData, 'JPEG', 0, 0, imgWidth, imgHeight);
          } else {
            while (leftHeight > 0) {
              PDF.addImage(pageData, 'JPEG', 0, position, imgWidth, imgHeight);
              leftHeight -= pageHeight;
              position -= 841.89;
              if (leftHeight > 0) {
                PDF.addPage();
              }
            }
          }
          // 导出
          PDF.save(paper?.name + '.pdf');
          resolve(true);
        })
        .catch(() => {
          reject(false);
        });
    });
  };
```

传入两个参数，1、title：转出的PDF文件的名字。2、dom：要进行转换的元素的id（把元素打上标记：id和class都行，因为要获取这个元素）

html2canvas参数：1、document.querySelector(dom)  获取的需要转换的元素

​									2、{

​												useCORS：true;     // 允许加载跨域资源

​												  allowTaint: true,    //允许画布被污染，跨域图片会污染画布

​												ignoreElements：(ele)=> boolean ;是否让当前元素画到画布上，true不允许，false允许

​												}

html2canvas方法调用后，产生一个回调，成功后的回调的参数就是canvas，然后转成base64编码。放大到PDF文件的大小，导出



```tsx
const isSplit = (nodes: NodeListOf<Element>, index: number, pageHeight: number) => {
    // 计算当前这块dom是否跨越了a4大小，以此分割，pageHeight：当前内容底部所在的纸加上之前的纸的总高度
    if (
      (nodes[index] as HTMLDivElement).offsetTop + (nodes[index] as HTMLDivElement).offsetHeight <
        pageHeight &&
      nodes[index + 1] &&
      (nodes[index + 1] as HTMLDivElement).offsetTop +
        (nodes[index + 1] as HTMLDivElement).offsetHeight >
        pageHeight
    ) {
      //当前元素的底部在这张纸上，接下来的这个元素的底部在下一张纸上
      return true;
    }
    return false;
  };
```

```tsx
const outPutPdfFn = () => {
    const target = document.querySelector('#outputpdf') as HTMLDivElement;
    const A4_WIDTH = 592.28;
    const A4_HEIGHT = 841.89;
    const pageHeight = (target.scrollWidth / A4_WIDTH) * A4_HEIGHT; //真实的一张纸的高度(按照比例缩放后的)
    // 获取分割dom，此处为class类名为item的dom
    const lableListID = document.querySelectorAll('.stemRender');
    // 进行分割操作，当dom内容已超出a4的高度，则将该dom前插入一个空dom，把他挤下去，分割
    for (let i = 0; i < lableListID.length; i++) {
      const lableItem = lableListID[i] as HTMLDivElement;
      const multiple = Math.ceil((lableItem.offsetTop + lableItem.offsetHeight) / pageHeight); //当前元素的底部在第几张纸上
      if (isSplit(lableListID, i, multiple * pageHeight)) {
        const divParent = lableListID[i].parentNode as HTMLDivElement; // 获取该div的父节点
        const newNode = document.createElement('div');
        newNode.className = 'emptyDiv';
        newNode.style.background = '#ffffff';
        const _H = multiple * pageHeight - (lableItem.offsetTop + lableItem.offsetHeight);  //当前元素的底部到当前的纸的底部的距离
        newNode.style.height = _H + 20 + 'px';   //因为下个元素是被分割的，上半部分就是的高度就是当前元素底部到当前纸底部的距离，+20是为了不仅靠最         //上面
        newNode.style.width = '100%';
        const next = lableListID[i].nextSibling; // 获取div的下一个兄弟节点
        // 判断兄弟节点是否存在

        if (next) {
          // 存在则将新节点插入到div的下一个兄弟节点之前，即div之后
          divParent.insertBefore(newNode, next);
        } else {
          // 不存在则直接添加到最后,appendChild默认添加到divParent的最后。这段代码无用，因为上面的切割判断必定是false，没有下个元素
          divParent.appendChild(newNode);
        }
      }
    }
    // 传入title和dom标签，此处是 #content
    // 异步函数，导出成功后处理交互
    getPdf('题目', '#outputpdf')
      .then(() => {
        // 自定义等待动画关闭
        setToPDFLoading(false);
        const emptyDivs = Array.from(document.querySelectorAll('.emptyDiv'));
        emptyDivs.forEach((item) => {
          (item as HTMLDivElement).remove();
        });

        message.info('试卷导出成功');
      })
      .catch(() => {
        setToPDFLoading(false);
        message.warn('试卷导出失败');
      });
  };
```

第一段代码是判断当前的元素时候要被分割（是否处于两个张PDF之间）；

第二段代码是将处于两张PDF之间的元素，向其头部添加一个只有高度的空元素.把这个元素撑下去

