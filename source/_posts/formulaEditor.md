---
title: 可交互的公式编辑器
date: 2023-07-25 11:27:15
tags:
- react-mathquill
- 在线编辑
- 公式
---

# 公式编辑器（可交互）

## 1、react-mathquill
{% dplayer "formula" url="/assets/video/formula.mp4"  custom="{width: '100px', height: '100px'}" %}
```npm
$ yarn add react-mathquill
$ yarn add antd
```

```tsx
import React, { useState, useEffect, useRef } from 'react';
import { addStyles, EditableMathField } from 'react-mathquill';

import type { MathField } from 'react-mathquill';
import { Button } from 'antd';

import { editData } from './editData';

// inserts the required css to the <head> block.
// you can skip this, if you want to do that by yourself.
addStyles();

const MyMathEditor = (props: {
  value: string;
  placeholder: string;
}) => {
  const {  placeholder, value } = props;
  const [latex, setLatex] = useState<string | undefined>(value);

  const [edit, setEdit] = useState<MathField>();
  const bthsRef = useRef<HTMLDivElement | null>(null);
  const placeholderRef = useRef<HTMLSpanElement | null>(null);
  let timer: NodeJS.Timeout | null = null;
  
  useEffect(() => {
    document.addEventListener('click', (e) => {
      const path = e.composedPath();
      if (
        !path.some(
          (r: any) =>
            r.className &&
            r.className.includes &&
            r.className.includes(`editMath`),
        )
      ) {
        (bthsRef.current as HTMLDivElement).style.display = 'none';
      }
    });
    return () => {
      document.removeEventListener('click', () => {});
    };
  }, []);

  useEffect(() => {}, [latex]);

  return (
    <div className={`editMath`} style={{ position: 'relative' }}>
      <EditableMathField
        className="edit"
        style={{ minWidth: '100px', border: 0 }}
        latex={latex}
        onBlur={() => {
         

          const myPlaceholder = placeholderRef.current as HTMLSpanElement;
          if (myPlaceholder) {
            myPlaceholder.style.display = 'inline';
          }
        }}
        onFocus={() => {
          if (timer) clearTimeout(timer);
          const myPlaceholder = placeholderRef.current;
          if (myPlaceholder) {
            myPlaceholder.style.display = 'none';
          }
          const allBtn = document.querySelectorAll('.editAllBtn');
          for (let i = 0; i < allBtn.length; i++) {
            const ele = allBtn[i] as HTMLButtonElement;
            ele.style.display = 'none';
          }

          const btn2 = bthsRef.current as HTMLDivElement;
          btn2.style.display = 'flex';

          edit?.focus();
        }}
        onInput={(e) => {
          const data = (e.target as any).value;
          (e.target as any).value = '';
          if (/^[\u4e00-\u9fa5]+$/.test(data)) {
            edit?.write(data);
            setLatex(edit?.latex());
          }
        }}
        onChange={(mathField) => {
          setLatex(mathField.latex());
        }}
        mathquillDidMount={(mathField) => {
          setEdit(mathField);
        }}
      />
      <span
        ref={(e) => (placeholderRef.current = e)}
        style={{
          position: 'absolute',
          top: '0px',
          left: '2px',
          zIndex: '1',
          pointerEvents: 'none',
          color: '#aaa',
        }}
      >
        {latex ? '' : placeholder}
      </span>
      <div
        ref={(e) => (bthsRef.current = e)}
        className={`editAllBtn`}
        style={{
          border: '1px solid #eee',
          width: '300px',
          display: 'none',
          justifyContent: 'flex-start',
          alignItems: 'flex-start',
          flexWrap: 'wrap',
          zIndex: '20',
          position: 'absolute',
          backgroundColor: '#fff',
          marginTop: '4px',
        }}
      >
        {editData.map((item) => (
          <Button
            tabIndex={-1}
            className="btn"
            onClick={(e) => {
              e.stopPropagation();

              edit?.focus();
              edit?.write(item.data);

              setLatex(edit?.latex());
              let theStep = 0;
              while (theStep < (item.step || 0)) {
                edit?.keystroke('Left');
                theStep++;
              }
            }}
            style={{
              width: '40px',
              height: '40px',
              display: 'flex',
              justifyContent: 'center',
              alignItems: 'center',
              margin: '10px',
            }}
            key={item.id}
          >
            <div dangerouslySetInnerHTML={{ __html: item.descri }} />
          </Button>
        ))}
      </div>
    </div>
  );
};

export { MyMathEditor };

```

react-mathquill会导出addStyles函数和EditableMathField组件。

在代码最顶层运行addStyles，加载公式编辑器所需要的全部样式。

EditableMathField：公式编辑器的组件(本代码对编辑器进行二次封装，实现了点击按钮和中文输入功能)

1. latex：latex公式的字符串，组件会将公式字符串渲染成可编辑的公式
2. onBlur：组件方法，编辑器失去焦点触发
3. onFocus：组件方法，编辑器获取焦点触发
4. mathquillDidMount：组件方法，当组件首次渲染时触发，自带参数为组件的实例对象
5. onChange：组件方法，组件重写的原生input中onChange方法，当编辑器中的公式改变时触发
6. onInput：继承自原生input中onInput方法，类似onChange（主要为了让编辑器支持中文输入）；

主要逻辑：

1. 在mathquillDidMount方法中获取组件的实例mathField，便于后面调用组件上的方法。当在编辑器中手动输入数据时候，会同时调用onChange和onInput方法，因为单个字母输入时，此时输入的值的长度为1，所以onInput后面不运行。当使用中文输入法输入时，此时输入的内容长度大于2，运行onInput，而当输入中文时，onChange又不运行（完美）
2. 当进行按钮快捷输入时，因为点在了编辑器以外的地方，造成其失去焦点，进而不能触发Button组件的onClick事件，解决方法是给按钮外层的div设置一个className，然后通过addEventListener事件监听每次的点击事件，获取此次点击元素的className，如果这些className里面不包含有那个div的className，就把元素的display设置成none，有就什么都不执行。所以在点击按钮时，因为有对应的classname，元素正常存在，触发onclick，先让编辑器聚焦（因为点击时候因为点击点不在编辑器上，会失去焦点），在使用edit?.write方法向编辑器写入公式。
3. 当点在其他地方时（非编辑器和按钮部分），没有对应的classname，元素失去焦点。
4. 编辑器聚焦时，获取所有与编辑器对应的外层div。把所有的div的display设成none，再使用ref获取当前点击的编辑器实例，把当前的设置成之前的flex；（效果就是其他的按钮都关了，就当前这个是开着的）；
5. 当编辑器失去焦点时，会向后端发送内容修改请求，如果此时，在100ms内又获取了这个编辑器的焦点，则请求不发送（因为在点击快捷输入按钮时，页面会经历一个快速的失焦，和聚焦的过程，此设置为了节流）；

注意:

1、上方是对库react-mathquill的二次封装，原本的react-mathquill库，不支持添加自定义的输入按钮，和中文输入

2、参数说明：(a)value:初始latex代码，如果没有需要传入一个空字符串（不传页面会显示undefind）；

​						 (b)placeholder:如果value值为空时，页面编辑器中显示的内容

​						 (c)onMyFinish:公式输入时，要执行的函数（需要修改成你自己的业务逻辑，向外界传数据）；

3、vite搭建的react项目，有个全局报错，不明原因；

4、editData是自定义按钮输入的数据，内容如下（把下方代码复制到editData.ts文件中，然后引入。最好放在上方组件的同级处）。

```ts
export const editData = [
  {
    id: 1,
    descri:
      '<svg xmlns="http://www.w3.org/2000/svg" width="1.76ex" height="1.505ex" viewBox="0 -583 778 665" xmlns:xlink="http://www.w3.org/1999/xlink" aria-hidden="true" style=""><defs><path id="MJX-344-TEX-N-2B" d="M56 237T56 250T70 270H369V420L370 570Q380 583 389 583Q402 583 409 568V270H707Q722 262 722 250T707 230H409V-68Q401 -82 391 -82H389H387Q375 -82 369 -68V230H70Q56 237 56 250Z"></path></defs><g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)"><g data-mml-node="math"><g data-mml-node="mo"><use xlink:href="#MJX-344-TEX-N-2B"></use></g></g></g></svg>',
    data: '+',
  },
  {
    id: 2,
    descri:
      '<svg xmlns="http://www.w3.org/2000/svg" width="1.76ex" height="1.505ex" viewBox="0 -583 778 665" xmlns:xlink="http://www.w3.org/1999/xlink" aria-hidden="true" style=""><defs><path id="MJX-350-TEX-N-2212" d="M84 237T84 250T98 270H679Q694 262 694 250T679 230H98Q84 237 84 250Z"></path></defs><g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)"><g data-mml-node="math"><g data-mml-node="mo"><use xlink:href="#MJX-350-TEX-N-2212"></use></g></g></g></svg>',
    data: '-',
  },
  {
    id: 3,
    descri:
      '<svg xmlns="http://www.w3.org/2000/svg" width="0.629ex" height="0.271ex" viewBox="0 -310 278 120" xmlns:xlink="http://www.w3.org/1999/xlink" aria-hidden="true" style=""><defs><path id="MJX-354-TEX-N-22C5" d="M78 250Q78 274 95 292T138 310Q162 310 180 294T199 251Q199 226 182 208T139 190T96 207T78 250Z"></path></defs><g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)"><g data-mml-node="math"><g data-mml-node="mo"><use xlink:href="#MJX-354-TEX-N-22C5"></use></g></g></g></svg>',
    data: '\\cdot',
  },
  {
    id: 4,
    descri:
      '<svg xmlns="http://www.w3.org/2000/svg" width="2.29ex" height="4.545ex" viewBox="0 -1118 1012 2009" xmlns:xlink="http://www.w3.org/1999/xlink" aria-hidden="true" style=""><defs><path id="MJX-364-TEX-I-1D465" d="M52 289Q59 331 106 386T222 442Q257 442 286 424T329 379Q371 442 430 442Q467 442 494 420T522 361Q522 332 508 314T481 292T458 288Q439 288 427 299T415 328Q415 374 465 391Q454 404 425 404Q412 404 406 402Q368 386 350 336Q290 115 290 78Q290 50 306 38T341 26Q378 26 414 59T463 140Q466 150 469 151T485 153H489Q504 153 504 145Q504 144 502 134Q486 77 440 33T333 -11Q263 -11 227 52Q186 -10 133 -10H127Q78 -10 57 16T35 71Q35 103 54 123T99 143Q142 143 142 101Q142 81 130 66T107 46T94 41L91 40Q91 39 97 36T113 29T132 26Q168 26 194 71Q203 87 217 139T245 247T261 313Q266 340 266 352Q266 380 251 392T217 404Q177 404 142 372T93 290Q91 281 88 280T72 278H58Q52 284 52 289Z"></path><path id="MJX-364-TEX-I-1D466" d="M21 287Q21 301 36 335T84 406T158 442Q199 442 224 419T250 355Q248 336 247 334Q247 331 231 288T198 191T182 105Q182 62 196 45T238 27Q261 27 281 38T312 61T339 94Q339 95 344 114T358 173T377 247Q415 397 419 404Q432 431 462 431Q475 431 483 424T494 412T496 403Q496 390 447 193T391 -23Q363 -106 294 -155T156 -205Q111 -205 77 -183T43 -117Q43 -95 50 -80T69 -58T89 -48T106 -45Q150 -45 150 -87Q150 -107 138 -122T115 -142T102 -147L99 -148Q101 -153 118 -160T152 -167H160Q177 -167 186 -165Q219 -156 247 -127T290 -65T313 -9T321 21L315 17Q309 13 296 6T270 -6Q250 -11 231 -11Q185 -11 150 11T104 82Q103 89 103 113Q103 170 138 262T173 379Q173 380 173 381Q173 390 173 393T169 400T158 404H154Q131 404 112 385T82 344T65 302T57 280Q55 278 41 278H27Q21 284 21 287Z"></path></defs><g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)"><g data-mml-node="math"><g data-mml-node="mfrac"><g data-mml-node="mi" transform="translate(220, 676)"><use xlink:href="#MJX-364-TEX-I-1D465"></use></g><g data-mml-node="mi" transform="translate(261, -686)"><use xlink:href="#MJX-364-TEX-I-1D466"></use></g><rect width="772" height="60" x="120" y="220"></rect></g></g></g></svg>',
    data: '\\frac{}{}',
    step: 1,
  },
  {
    id: 5,
    descri:
      '<svg xmlns="http://www.w3.org/2000/svg" width="3.224ex" height="2.398ex" viewBox="0 -890.8 1425 1060" xmlns:xlink="http://www.w3.org/1999/xlink" aria-hidden="true" style=""><defs><path id="MJX-374-TEX-N-221A" d="M95 178Q89 178 81 186T72 200T103 230T169 280T207 309Q209 311 212 311H213Q219 311 227 294T281 177Q300 134 312 108L397 -77Q398 -77 501 136T707 565T814 786Q820 800 834 800Q841 800 846 794T853 782V776L620 293L385 -193Q381 -200 366 -200Q357 -200 354 -197Q352 -195 256 15L160 225L144 214Q129 202 113 190T95 178Z"></path><path id="MJX-374-TEX-I-1D465" d="M52 289Q59 331 106 386T222 442Q257 442 286 424T329 379Q371 442 430 442Q467 442 494 420T522 361Q522 332 508 314T481 292T458 288Q439 288 427 299T415 328Q415 374 465 391Q454 404 425 404Q412 404 406 402Q368 386 350 336Q290 115 290 78Q290 50 306 38T341 26Q378 26 414 59T463 140Q466 150 469 151T485 153H489Q504 153 504 145Q504 144 502 134Q486 77 440 33T333 -11Q263 -11 227 52Q186 -10 133 -10H127Q78 -10 57 16T35 71Q35 103 54 123T99 143Q142 143 142 101Q142 81 130 66T107 46T94 41L91 40Q91 39 97 36T113 29T132 26Q168 26 194 71Q203 87 217 139T245 247T261 313Q266 340 266 352Q266 380 251 392T217 404Q177 404 142 372T93 290Q91 281 88 280T72 278H58Q52 284 52 289Z"></path></defs><g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)"><g data-mml-node="math"><g data-mml-node="msqrt"><g transform="translate(853, 0)"><g data-mml-node="mi"><use xlink:href="#MJX-374-TEX-I-1D465"></use></g></g><g data-mml-node="mo" transform="translate(0, 30.8)"><use xlink:href="#MJX-374-TEX-N-221A"></use></g><rect width="572" height="60" x="853" y="770.8"></rect></g></g></g></svg>',
    data: '\\sqrt{}',
    step: 1,
  },
  {
    id: 6,
    descri:
      '<svg xmlns="http://www.w3.org/2000/svg" width="3.224ex" height="2.398ex" viewBox="0 -890.8 1425 1060" xmlns:xlink="http://www.w3.org/1999/xlink" aria-hidden="true" style=""><defs><path id="MJX-384-TEX-N-33" d="M127 463Q100 463 85 480T69 524Q69 579 117 622T233 665Q268 665 277 664Q351 652 390 611T430 522Q430 470 396 421T302 350L299 348Q299 347 308 345T337 336T375 315Q457 262 457 175Q457 96 395 37T238 -22Q158 -22 100 21T42 130Q42 158 60 175T105 193Q133 193 151 175T169 130Q169 119 166 110T159 94T148 82T136 74T126 70T118 67L114 66Q165 21 238 21Q293 21 321 74Q338 107 338 175V195Q338 290 274 322Q259 328 213 329L171 330L168 332Q166 335 166 348Q166 366 174 366Q202 366 232 371Q266 376 294 413T322 525V533Q322 590 287 612Q265 626 240 626Q208 626 181 615T143 592T132 580H135Q138 579 143 578T153 573T165 566T175 555T183 540T186 520Q186 498 172 481T127 463Z"></path><path id="MJX-384-TEX-N-221A" d="M95 178Q89 178 81 186T72 200T103 230T169 280T207 309Q209 311 212 311H213Q219 311 227 294T281 177Q300 134 312 108L397 -77Q398 -77 501 136T707 565T814 786Q820 800 834 800Q841 800 846 794T853 782V776L620 293L385 -193Q381 -200 366 -200Q357 -200 354 -197Q352 -195 256 15L160 225L144 214Q129 202 113 190T95 178Z"></path><path id="MJX-384-TEX-I-1D465" d="M52 289Q59 331 106 386T222 442Q257 442 286 424T329 379Q371 442 430 442Q467 442 494 420T522 361Q522 332 508 314T481 292T458 288Q439 288 427 299T415 328Q415 374 465 391Q454 404 425 404Q412 404 406 402Q368 386 350 336Q290 115 290 78Q290 50 306 38T341 26Q378 26 414 59T463 140Q466 150 469 151T485 153H489Q504 153 504 145Q504 144 502 134Q486 77 440 33T333 -11Q263 -11 227 52Q186 -10 133 -10H127Q78 -10 57 16T35 71Q35 103 54 123T99 143Q142 143 142 101Q142 81 130 66T107 46T94 41L91 40Q91 39 97 36T113 29T132 26Q168 26 194 71Q203 87 217 139T245 247T261 313Q266 340 266 352Q266 380 251 392T217 404Q177 404 142 372T93 290Q91 281 88 280T72 278H58Q52 284 52 289Z"></path></defs><g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)"><g data-mml-node="math"><g data-mml-node="mroot"><g><g data-mml-node="mi" transform="translate(853, 0)"><use xlink:href="#MJX-384-TEX-I-1D465"></use></g></g><g data-mml-node="mn" transform="translate(261.8, 391.8) scale(0.5)"><use xlink:href="#MJX-384-TEX-N-33"></use></g><g data-mml-node="mo" transform="translate(0, 30.8)"><use xlink:href="#MJX-384-TEX-N-221A"></use></g><rect width="572" height="60" x="853" y="770.8"></rect></g></g></g></svg>',
    data: '\\sqrt[3]{}',
    step: 1,
  },
  {
    id: 7,
    descri:
      '<svg xmlns="http://www.w3.org/2000/svg" width="1.996ex" height="2.067ex" viewBox="0 -903.7 882.3 913.7" xmlns:xlink="http://www.w3.org/1999/xlink" aria-hidden="true" style=""><defs><path id="MJX-424-TEX-I-1D44E" d="M33 157Q33 258 109 349T280 441Q331 441 370 392Q386 422 416 422Q429 422 439 414T449 394Q449 381 412 234T374 68Q374 43 381 35T402 26Q411 27 422 35Q443 55 463 131Q469 151 473 152Q475 153 483 153H487Q506 153 506 144Q506 138 501 117T481 63T449 13Q436 0 417 -8Q409 -10 393 -10Q359 -10 336 5T306 36L300 51Q299 52 296 50Q294 48 292 46Q233 -10 172 -10Q117 -10 75 30T33 157ZM351 328Q351 334 346 350T323 385T277 405Q242 405 210 374T160 293Q131 214 119 129Q119 126 119 118T118 106Q118 61 136 44T179 26Q217 26 254 59T298 110Q300 114 325 217T351 328Z"></path><path id="MJX-424-TEX-I-1D44F" d="M73 647Q73 657 77 670T89 683Q90 683 161 688T234 694Q246 694 246 685T212 542Q204 508 195 472T180 418L176 399Q176 396 182 402Q231 442 283 442Q345 442 383 396T422 280Q422 169 343 79T173 -11Q123 -11 82 27T40 150V159Q40 180 48 217T97 414Q147 611 147 623T109 637Q104 637 101 637H96Q86 637 83 637T76 640T73 647ZM336 325V331Q336 405 275 405Q258 405 240 397T207 376T181 352T163 330L157 322L136 236Q114 150 114 114Q114 66 138 42Q154 26 178 26Q211 26 245 58Q270 81 285 114T318 219Q336 291 336 325Z"></path></defs><g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)"><g data-mml-node="math"><g data-mml-node="msup"><g data-mml-node="mi"><use xlink:href="#MJX-424-TEX-I-1D44E"></use></g><g data-mml-node="TeXAtom" transform="translate(529, 413) scale(0.707)" data-mjx-texclass="ORD"><g data-mml-node="mi"><use xlink:href="#MJX-424-TEX-I-1D44F"></use></g></g></g></g></g></svg>',
    data: 'a^{b}',
    step: 1,
  },
  {
    id: 8,
    descri:
      '<svg xmlns="http://www.w3.org/2000/svg" width="1.29ex" height="1ex" viewBox="0 -431 570 442" xmlns:xlink="http://www.w3.org/1999/xlink" aria-hidden="true" style=""><defs><path id="MJX-428-TEX-I-1D70B" d="M132 -11Q98 -11 98 22V33L111 61Q186 219 220 334L228 358H196Q158 358 142 355T103 336Q92 329 81 318T62 297T53 285Q51 284 38 284Q19 284 19 294Q19 300 38 329T93 391T164 429Q171 431 389 431Q549 431 553 430Q573 423 573 402Q573 371 541 360Q535 358 472 358H408L405 341Q393 269 393 222Q393 170 402 129T421 65T431 37Q431 20 417 5T381 -10Q370 -10 363 -7T347 17T331 77Q330 86 330 121Q330 170 339 226T357 318T367 358H269L268 354Q268 351 249 275T206 114T175 17Q164 -11 132 -11Z"></path></defs><g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)"><g data-mml-node="math"><g data-mml-node="mi"><use xlink:href="#MJX-428-TEX-I-1D70B"></use></g></g></g></svg>',
    data: '\\pi',
  },
  {
    id: 9,
    descri:
      '<svg xmlns="http://www.w3.org/2000/svg" width="1.76ex" height="1.09ex" viewBox="0 -491 778 482" xmlns:xlink="http://www.w3.org/1999/xlink" aria-hidden="true" style=""><defs><path id="MJX-434-TEX-N-D7" d="M630 29Q630 9 609 9Q604 9 587 25T493 118L389 222L284 117Q178 13 175 11Q171 9 168 9Q160 9 154 15T147 29Q147 36 161 51T255 146L359 250L255 354Q174 435 161 449T147 471Q147 480 153 485T168 490Q173 490 175 489Q178 487 284 383L389 278L493 382Q570 459 587 475T609 491Q630 491 630 471Q630 464 620 453T522 355L418 250L522 145Q606 61 618 48T630 29Z"></path></defs><g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)"><g data-mml-node="math"><g data-mml-node="mo"><use xlink:href="#MJX-434-TEX-N-D7"></use></g></g></g></svg>',
    data: '\\times',
  },
  {
    id: 10,
    descri:
      '<svg xmlns="http://www.w3.org/2000/svg" width="1.76ex" height="1.296ex" viewBox="0 -537 778 573" xmlns:xlink="http://www.w3.org/1999/xlink" aria-hidden="true" style=""><defs><path id="MJX-438-TEX-N-F7" d="M318 466Q318 500 339 518T386 537Q418 537 438 517T458 466Q458 438 440 417T388 396Q355 396 337 417T318 466ZM56 237T56 250T70 270H706Q721 262 721 250T706 230H70Q56 237 56 250ZM318 34Q318 68 339 86T386 105Q418 105 438 85T458 34Q458 6 440 -15T388 -36Q355 -36 337 -15T318 34Z"></path></defs><g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)"><g data-mml-node="math"><g data-mml-node="TeXAtom" data-mjx-texclass="ORD"><g data-mml-node="mo"><use xlink:href="#MJX-438-TEX-N-F7"></use></g></g></g></g></svg>',
    data: '{\\div}',
  },
  {
    id: 11,
    descri:
      '<svg xmlns="http://www.w3.org/2000/svg" width="3.092ex" height="5.616ex" viewBox="0 -1578.8 1366.7 2482.2" xmlns:xlink="http://www.w3.org/1999/xlink" aria-hidden="true" style=""><defs><path id="MJX-452-TEX-LO-222B" d="M114 -798Q132 -824 165 -824H167Q195 -824 223 -764T275 -600T320 -391T362 -164Q365 -143 367 -133Q439 292 523 655T645 1127Q651 1145 655 1157T672 1201T699 1257T733 1306T777 1346T828 1360Q884 1360 912 1325T944 1245Q944 1220 932 1205T909 1186T887 1183Q866 1183 849 1198T832 1239Q832 1287 885 1296L882 1300Q879 1303 874 1307T866 1313Q851 1323 833 1323Q819 1323 807 1311T775 1255T736 1139T689 936T633 628Q574 293 510 -5T410 -437T355 -629Q278 -862 165 -862Q125 -862 92 -831T55 -746Q55 -711 74 -698T112 -685Q133 -685 150 -700T167 -741Q167 -789 114 -798Z"></path><path id="MJX-452-TEX-I-1D44F" d="M73 647Q73 657 77 670T89 683Q90 683 161 688T234 694Q246 694 246 685T212 542Q204 508 195 472T180 418L176 399Q176 396 182 402Q231 442 283 442Q345 442 383 396T422 280Q422 169 343 79T173 -11Q123 -11 82 27T40 150V159Q40 180 48 217T97 414Q147 611 147 623T109 637Q104 637 101 637H96Q86 637 83 637T76 640T73 647ZM336 325V331Q336 405 275 405Q258 405 240 397T207 376T181 352T163 330L157 322L136 236Q114 150 114 114Q114 66 138 42Q154 26 178 26Q211 26 245 58Q270 81 285 114T318 219Q336 291 336 325Z"></path><path id="MJX-452-TEX-I-1D44E" d="M33 157Q33 258 109 349T280 441Q331 441 370 392Q386 422 416 422Q429 422 439 414T449 394Q449 381 412 234T374 68Q374 43 381 35T402 26Q411 27 422 35Q443 55 463 131Q469 151 473 152Q475 153 483 153H487Q506 153 506 144Q506 138 501 117T481 63T449 13Q436 0 417 -8Q409 -10 393 -10Q359 -10 336 5T306 36L300 51Q299 52 296 50Q294 48 292 46Q233 -10 172 -10Q117 -10 75 30T33 157ZM351 328Q351 334 346 350T323 385T277 405Q242 405 210 374T160 293Q131 214 119 129Q119 126 119 118T118 106Q118 61 136 44T179 26Q217 26 254 59T298 110Q300 114 325 217T351 328Z"></path></defs><g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)"><g data-mml-node="math"><g data-mml-node="msubsup"><g data-mml-node="mo"><use xlink:href="#MJX-452-TEX-LO-222B"></use></g><g data-mml-node="TeXAtom" transform="translate(1013.4, 1088.1) scale(0.707)" data-mjx-texclass="ORD"><g data-mml-node="mi"><use xlink:href="#MJX-452-TEX-I-1D44F"></use></g></g><g data-mml-node="TeXAtom" transform="translate(556, -896.4) scale(0.707)" data-mjx-texclass="ORD"><g data-mml-node="mi"><use xlink:href="#MJX-452-TEX-I-1D44E"></use></g></g></g></g></g></svg>',
    data: '\\int_{a}^{b}',
    step: 1,
  },
];

```

