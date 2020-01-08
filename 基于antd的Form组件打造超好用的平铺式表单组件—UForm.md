# 基于antd Form组件打造超好用的平铺式表单组件—UForm

## 前言
博主最近在用antd Form组件开发表单功能，深深感慨antd Form组件功能也忒强大了，但是写起来真的挺麻烦的，相信很多小伙伴在用antd Form组件的时候都有同样的感受吧。

看一下antd官网的一个例子，需要Form.create包装表单组件，然后表单每一项使用getFieldDecorator进行具体描述。
```
import { Form, Input, Button, Checkbox } from 'antd';

const formItemLayout = {
  labelCol: { span: 4 },
  wrapperCol: { span: 8 },
};
const formTailLayout = {
  labelCol: { span: 4 },
  wrapperCol: { span: 8, offset: 4 },
};
class DynamicRule extends React.Component {
  state = {
    checkNick: false,
  };

  check = () => {
    this.props.form.validateFields(err => {
      if (!err) {
        console.info('success');
      }
    });
  };

  handleChange = e => {
    this.setState(
      {
        checkNick: e.target.checked,
      },
      () => {
        this.props.form.validateFields(['nickname'], { force: true });
      },
    );
  };

  render() {
    const { getFieldDecorator } = this.props.form;
    return (
      <div>
        <Form.Item {...formItemLayout} label="Name">
          {getFieldDecorator('username', {
            rules: [
              {
                required: true,
                message: 'Please input your name',
              },
            ],
          })(<Input placeholder="Please input your name" />)}
        </Form.Item>
        <Form.Item {...formItemLayout} label="Nickname">
          {getFieldDecorator('nickname', {
            rules: [
              {
                required: this.state.checkNick,
                message: 'Please input your nickname',
              },
            ],
          })(<Input placeholder="Please input your nickname" />)}
        </Form.Item>
        <Form.Item {...formTailLayout}>
          <Checkbox checked={this.state.checkNick} onChange={this.handleChange}>
            Nickname is required
          </Checkbox>
        </Form.Item>
        <Form.Item {...formTailLayout}>
          <Button type="primary" onClick={this.check}>
            Check
          </Button>
        </Form.Item>
      </div>
    );
  }
}

const WrappedDynamicRule = Form.create({ name: 'dynamic_rule' })(DynamicRule);
ReactDOM.render(<WrappedDynamicRule />, mountNode);
```
**思考**：能不能只传data和fields字段，底层不需要关心表单项的生成过程呢。
## 基本介绍
于是，博主费尽千辛万苦封装了一个超好用的UForm组件，只需要传入data、fields props和onChange、onCancel、onSubmit回调即可渲染一个完整的表单了，支持**表单渲染**、**表单验证**、**修改回调**、**提交回调**、**取消回调**。
```
<UForm
    data={data}
    fields={fields}
    onChange={onChange}
    onCancel={onCancel}
    onSubmit={onSubmit}
    />
```
目前UForm组件支持**常规表单**及**弹窗表单**两种，支持Form组件包含`Input`、`InputNumber`、`Textarea`、`Select`、`Radio`、`Checkbox`、`Password`、`Switch`、`Rate`、`custom（自定义ReactNode）`。

**documentation：[https://dadaiwei.github.io/uform](https://dadaiwei.github.io/uform)**
<br>
**npm：[https://www.npmjs.com/package/uform](https://www.npmjs.com/package/uform)**

## 使用指南
1.安装 UForm 依赖包

```
npm install uform --save-dev
```

2.引入依赖包

```
import UForm from "uform";
import "uform/dist/uform.css";
```

3.使用 UForm 组件

```
<UForm
    data={data}
    fields={fields}
    onChange={onChange}
    onCancel={onCancel}
    onSubmit={onSubmit}
    />
```
## 代码演示

### 通用表单

![通用表单](https://user-gold-cdn.xitu.io/2019/9/2/16ceff603993e45f?w=1208&h=773&f=png&s=43212)

代码

```
import React, { useState } from "react";
import UForm from "uform";
import "uform/dist/uform.css";

function NormalForm() {
  const [data, setData] = useState({
    name: "",
    size: 1,
    city: 0,
    area: "郊区",
    password: "",
    choosen: true,
    confirm: true,
    rate: 3,
    describe: ""
  });
  const fields = [
    {
      name: "name",
      label: "名称",
      type: "input",
      placeholder: "请输入名称",
      rules: [
        {
          required: true,
          message: "请输入名称"
        }
      ]
    },
    {
      name: "size",
      label: "大小",
      type: "inputNumber",
      placeholder: "请输入大小",
      rules: [
        {
          required: true,
          message: "请输入名称"
        }
      ]
    },
    {
      name: "city",
      label: "城市",
      type: "select",
      fieldData: [
        {
          name: "北京",
          value: 0
        },
        {
          name: "上海",
          value: 1
        },
        {
          name: "杭州",
          value: 2
        },
        {
          name: "深圳",
          value: 3
        }
      ],
      rules: [
        {
          required: true,
          message: "请输入名称"
        }
      ]
    },
    {
      name: "area",
      label: "地区",
      type: "radio",
      fieldData: ["城区", "郊区"]
    },
    {
      name: "confirm",
      label: "确认选择",
      type: "checkbox",
      rules: [
        {
          required: true,
          message: "请确认选择"
        }
      ]
    },
    {
      name: "custom",
      label: "自定义项",
      type: "custom",
      node: (
        <div>
          <h2>自定义表单项</h2>
        </div>
      )
    },
    {
      name: "password",
      label: "密码",
      type: "password",
      rules: [
        {
          required: true,
          message: "请输入密码"
        }
      ]
    },
    {
      name: "choosen",
      label: "是否选择",
      type: "switch",
      checkedChildren: "开",
      unCheckedChildren: "关",
      rules: [
        {
          required: true,
          message: "请输入密码"
        }
      ]
    },
    {
      name: "rate",
      label: "评分",
      type: "rate"
    },
    {
      name: "describe",
      label: "描述",
      type: "textarea",
      placeholder: "请输入描述"
    }
  ];
  const onChange = (result) => {
    setData({
      ...data,
      ...result
    });
  };
  const onSubmit = (values) => {
    console.log(values);
  };
  const onCancel = () => {
    console.log("cancel");
  };
  return (
    <div>
      <div style={{ width: 600, marginLeft: 100 }}>
        <UForm
          data={data}
          fields={fields}
          onChange={onChange}
          onCancel={onCancel}
          onSubmit={onSubmit}
        />
      </div>
    </div>
  );
}
```

### 弹框表单

![谈框表单](https://user-gold-cdn.xitu.io/2019/9/2/16ceffbbcf8ce807?w=774&h=782&f=png&s=46046)

代码

```
import React, { useState } from "react";
import { Button } from "antd";
import UForm from "uform";
import "uform/dist/uform.css";

function ModalForm() {
  const { visible, setVisible } = useState(false);
  const [data, setData] = useState({
    name: "",
    size: 1,
    city: 0,
    area: "郊区",
    password: "",
    choosen: true,
    confirm: true,
    rate: 3,
    describe: ""
  });
  const fields = [
    {
      name: "name",
      label: "名称",
      type: "input",
      placeholder: "请输入名称",
      rules: [
        {
          required: true,
          message: "请输入名称"
        }
      ]
    },
    {
      name: "size",
      label: "大小",
      type: "inputNumber",
      placeholder: "请输入大小",
      rules: [
        {
          required: true,
          message: "请输入名称"
        }
      ]
    },
    {
      name: "city",
      label: "城市",
      type: "select",
      fieldData: [
        {
          name: "北京",
          value: 0
        },
        {
          name: "上海",
          value: 1
        },
        {
          name: "杭州",
          value: 2
        },
        {
          name: "深圳",
          value: 3
        }
      ],
      rules: [
        {
          required: true,
          message: "请输入名称"
        }
      ]
    },
    {
      name: "area",
      label: "地区",
      type: "radio",
      fieldData: ["城区", "郊区"]
    },
    {
      name: "confirm",
      label: "确认选择",
      type: "checkbox",
      rules: [
        {
          required: true,
          message: "请确认选择"
        }
      ]
    },
    {
      name: "custom",
      label: "自定义项",
      type: "custom",
      node: (
        <div>
          <h2>自定义表单项</h2>
        </div>
      )
    },
    {
      name: "password",
      label: "密码",
      type: "password",
      rules: [
        {
          required: true,
          message: "请输入密码"
        }
      ]
    },
    {
      name: "choosen",
      label: "是否选择",
      type: "switch",
      checkedChildren: "开",
      unCheckedChildren: "关",
      rules: [
        {
          required: true,
          message: "请输入密码"
        }
      ]
    },
    {
      name: "rate",
      label: "评分",
      type: "rate"
    },
    {
      name: "describe",
      label: "描述",
      type: "textarea",
      placeholder: "请输入描述"
    }
  ];
  const onChange = (result) => {
    setData({
      ...data,
      ...result
    });
  };
  const onSubmit = (values) => {
    console.log(values);
  };
  const onCancel = () => {
    setVisible(false);
  };
  return (
    <div>
      <Button
        type='primary'
        onClick={() => {
          setVisible(true);
        }}>
        打开弹窗表单
      </Button>
      <div style={{ width: 600, marginLeft: 100 }}>
        <UForm
          type="modal"
          visible={visible}
          title='新建表单'
          data={data}
          fields={fields}
          onChange={onChange}
          onCancel={onCancel}
          onSubmit={onSubmit}
        />
      </div>
    </div>
  );
}
```

## API

### 公共 API

| 属性       | 说明                     | 必填属性 | 类型             | 可选值                            | 默认值     |
| :--------- | :----------------------- | :------- | :--------------- | :-------------------------------- | :--------- |
| type       | 使用通用表单或者弹框表单 | false    | string           | norma \| modal                    | normal     |
| layout     | 表单布局                 | false    | string           | horizontal \| vertical \| inline  | horizontal |
| labelCol   | 表单 label 占宽          | false    | number           | 1-24 之间整数                     | 4          |
| wrapperCol | 表单内容项占宽           | false    | number           | 1-24 之间整数，通常为 24-labelCol | 16         |
| loading    | 确定按钮 loading         | false    | boolean          | true \| false                     | false      |
| data       | 表单数据                 | true     | any[ ]           | -                                 | [ ]        |
| fields     | 表单每一项特征描述       | true     | any[ ]           | -                                 | [ ]        |
| onSubmit   | 表单提交回调             | true     | Function(values) | -                                 | 无         |
| onChange   | 表单每一项修改回调       | false    | Function(value)  | -                                 | 无         |
| onCancel   | 表单取消回调             | false    | Function( )      | -                                 | 无         |

### fields props

| 属性       | 说明                                            | 必填属性 | 类型       | 可选值                                                                                                              | 默认值 |
| :--------- | :---------------------------------------------- | :------- | :--------- | :------------------------------------------------------------------------------------------------------------------ | :----- |
| name       | 表单数据 name，与 data 中 key 值一致            | true     | string     | -                                                                                                                   | 无     |
| type       | 表单项类型                                      | true     | string     | input \| inputNumber \| textarea \| select \| radio \| checkbox \| password \| switch \| rate \| custom（自定义项） | 无     |
| label      | 表单项 label                                    | true     | string     | -                                                                                                                   | 无     |
| rules      | 表单项限制规则，与 antd Form 限制规则一致       | false    | object [ ] | -                                                                                                                   | 无     |
| fieldData  | 适用于 type 为 select/radio 的选择项            | true     | object [ ] | -                                                                                                                   | 无     |
| placehoder | 适用于 type 为 input/textarea/select 的提示信息 | false    | string     | -                                                                                                                   | 无     |
| node       | 适用于 type 为 custom 的自定义 ReactNode        | true     | ReactNode  | -                                                                                                                   | 无     |

> **特别说明**：type 类型为 **select/radio**时， **fieldsData** 是必填属性， type 类型为 **custome** 时， **node** 是必填属性。

### 通用表单特有API

| 属性             | 说明             | 必填属性 | 类型    | 可选值        | 默认值 |
| :--------------- | :--------------- | :------- | :------ | :------------ | :----- |
| showCancelButton | 是否展示取消按钮 | false    | boolean | true \| false | true   |

### 弹框表单特有API

| 属性    | 说明         | 必填属性 | 类型                | 可选值        | 默认值 |
| :------ | :----------- | :------- | :------------------ | :------------ | :----- |
| visible | 弹框是否可见 | true     | boolean             | true \| false | true   |
| title   | 弹框标题     | true     | string \| ReactNode | -             | 无     |
| width   | 弹框宽度     | false    | number              | -             | 600    |

## 总结
以上就是博主基于antd Form组件封装的UForm组件，目前支持功能还比较简单，后续会逐渐完善扩展该组件的功能，感兴趣的小伙伴可以试用下，真的简单又好用。

*觉得文章不错的小伙伴可以点个赞，灰常感谢^_^。*
