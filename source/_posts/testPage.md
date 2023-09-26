---

title: 页面转pdf进阶（渲染任意样式）

date: 2023-09-26 11:05:52

tags:
- 带封条试卷
- html2canvas
- jspdf
---

# 页面竖排的题目转成考试时，带有封条的试卷

## 1、html2canvas&&jspdf
![试卷效果](/assets/images/testPage.png)
```
$ yarn add html2canvas jspdf
```

```
import html2canvas from 'html2canvas';
import JsPDF from 'jspdf';
```
页面渲染出的封条（和后面创建的pdf高度保持一致）
```tsx
 <div className={styles.sealHidden}>
        <div ref={(e) => (sealRef.current = e as HTMLDivElement)} className={styles.seal}>
          <div className={styles.sealSide}>
            ┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;○&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;外&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;○&nbsp;
            ┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;装&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;○&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;订&nbsp;
            ┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;○&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;线&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;○&nbsp;
            ┈&nbsp;┈&nbsp;┈&nbsp;┈
          </div>
          <div className={styles.sealcontent}>
            <div className={styles.sealItemSide} />
            <div className={styles.sealItemContent}>
              学校:___________姓名:___________班级:___________考号:___________
            </div>
            <div className={styles.sealItemSide} />
          </div>
          <div className={styles.sealSideLast}>
            ┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;○&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;外&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;○&nbsp;
            ┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;装&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;○&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;订&nbsp;
            ┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;○&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;线&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;┈&nbsp;○&nbsp;
            ┈&nbsp;┈&nbsp;┈&nbsp;┈
          </div>
        </div>
      </div>
```
确定页面封条渲染完成后，转成图片的base64编码存起来
```tsx
useEffect(() => {
    html2canvas(sealRef.current as HTMLDivElement, {
      useCORS: true,
      allowTaint: true,
    }).then((canvas) => {
      sealdataRef.current = canvas.toDataURL('image/jpeg', 1.0);
    });
  }, [sealRef.current]);
```
```tsx
const outPutTestPdfFn = () => {
    return new Promise((resolve, reject) => {
      html2canvas(document.querySelector('#outputpdf') as HTMLDivElement, {
        useCORS: true,   //允许页面被污染
        allowTaint: true,  //允许页面图片跨域
        ignoreElements: (ele) => {    //ele是#outputpdf的真实dom，此方法遍历ele里所有元素，返回true则被删除
          if (ele.className === 'blanks') {
            return true;
          }
          return false;
        },
      })
        .then(function (canvas) {
          //canvas是页面拿#outputpdf生成的（#outputpdf是页面要被转pdf的元素）
          const realPageHeight = 841.89; //一张a4纸的高度
          const realPageWidth = 592.28; //一张a4纸的宽度
          const sealWidth = 80.667; //封条的宽度
          const contentWidth = canvas.width;  
          const contentHeight = canvas.height;
          // 根据A4纸的大小，计算出dom相应比例的尺寸
          const pageHeight = (contentWidth / realPageWidth) * realPageHeight;  //(元素宽)/纸宽*纸高=a4一页的高度
          let leftHeight = contentHeight;
          let position = 0;  //用于img从哪个高度开始渲染（图片最上面高度是0，往下是负数）
          const imgWidth = realPageWidth;
          // 根据a4比例计算出需要分割的实际dom位置
          const imgHeight = (realPageWidth / contentWidth) * contentHeight;  //img按比例缩放到和a4一样（仅宽度）
          // canvas绘图生成image数据，1.0是质量参数
          const pageData = canvas.toDataURL('image/jpeg', 1.0);
          // 试卷大小
          const PDF = new JsPDF({
            unit: 'px',
            orientation: 'l',
            format: [realPageWidth * 2 + sealWidth, realPageHeight],
          }); //创建a4，宽度就是封条加两个图片
          if (leftHeight < pageHeight * 2) {   //图片高度不超过两张纸情况（也就是当前页，因为两份被渲染在一页上）
          // addImage(imageData, format, pdf的x轴位置（y轴默认为0）, img的y轴位置（x轴默认为0） , imgWidth, imgHeight,...)；
            PDF.addImage(sealdataRef.current as string, 'JPEG', 0, 0, sealWidth, realPageHeight);
            PDF.addImage(pageData, 'JPEG', sealWidth, 0, imgWidth, imgHeight);
            PDF.addImage(pageData, 'JPEG', sealWidth + imgWidth, 0, imgWidth, imgHeight);
          } else {
            while (leftHeight > 0) {
              PDF.addImage(sealdataRef.current as string, 'JPEG', 0, 0, sealWidth, realPageHeight); //封条
              PDF.addImage(pageData, 'JPEG', sealWidth, position, imgWidth, imgHeight); //左边内容

              PDF.addImage(
                pageData,
                'JPEG',
                sealWidth + imgWidth,
                position - realPageHeight,
                imgWidth,
                imgHeight,
              ); //右边内容
              leftHeight -= pageHeight * 2;
              position -= realPageHeight * 2;
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

如果向在任意位置渲染任意内容,可将pageHeight设置为正值，图片会被向下挤，留出上方空白部分


