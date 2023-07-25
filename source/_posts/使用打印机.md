---

title: 特别简单的打印机使用

date: 2023-07-25 13:45:52

tags:
- 打印
- react-to-print
---

# 打印文件

## 1、react-to-print

```
$ yarn add react-to-print
```

```
import ReactToPrint from 'react-to-print';
```

```tsx
 <ReactToPrint
                  trigger={() => (
                    <Button icon={<PrinterOutlined />} onClick={() => printRef?.current}>
                      打印
                    </Button>
                  )}
                  content={() => printRef.current || null}
                />
```

直接默认导出一个组件，content是一个函数，返回值是组件的真实DOM(用useRef获取)。trigger作为函数返回一个在页面上显示此功能的标签

