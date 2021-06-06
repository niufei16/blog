---
title: CodeIgniter 4 开发（九）
date: 2021-06-06 17:17:47
tags:
	- php
	- CodeIgniter
---
> 图像裁剪



图像裁剪的相关方法请参考文档[图像处理类](https://codeigniter.org.cn/user_guide/libraries/images.html)

## 辅助函数

新建图像裁剪的辅助函数
app/Helpers/imageUpload_helper.php

```php
<?php
if (!function_exists('save_image_upload'))
{
  function save_image_upload($file, $folder, $file_prefix, $max_width, $max_height)
  {
    $ext = $file->getClientExtension();

    helper('text');
    $fileName = date('Y-m-d') . '_' . $file_prefix .'_' . random_string() .'.' . $ext;
    $path = FCPATH . 'image/' . $folder;

    if ( !$file->move($path, $fileName, true))
    {
      return false;
    }

    if ($max_width && $max_height && $ext !== 'gif')
    {
      \Config\Services::image()
        ->withFile($path . '/' . $fileName)
        ->resize($max_width, $max_height)
        ->save($path . '/' . $fileName);
    }

    return [
      'path' => 'image/' . $folder . '/' . $fileName,
    ];
  }
}
```
## 上传

修改头像上传时的逻辑
app/Controllers/Auth/UsersController.php

```php
public function avatarUpload($id)
{
    $rules = [
        'avatar' => [
            'label' => '头像',
            'rules' => 'uploaded[avatar]|max_size[avatar,200]|ext_in[avatar,jpg,png,gif]',
        ]
    ];

    if (!$this->validate($rules))
    {
        return redirect()->back()->with('errors', $this->validator->getErrors());
    }

    $file = $this->request->getFile('avatar');

    if (!$file->isValid() || $file->hasMoved())
    {
        return redirect()->back()->with('error', $file->getError());
    }

-    $ext = $file->getClientExtension();
-
-    helper('text');
-    $fileName = date('Y-m-d') . '_' . $id .'_' . random_string() .'.' . $ext;
-    $path = FCPATH . 'image\avatar';
-
-    if ( $file->move($path, $fileName, true))
-    {
-        $user = [
-            'avatar' => 'avatar/' . $fileName,
-        ];
-
-        $this->userModel->update($id, $user);
-
-        return redirect()->back()->with('success', '头像上传成功~');
-    }
-
-    return redirect()->back()->with('error', '头像上传失败~');
    
+    helper('imageUpload');
+    $result = save_image_upload($file, 'avatar', $id, 512, 512);
+
+    if (!$result)
+    {
+      return redirect()->back()->with('error', '头像上传失败~');
+    }
+
+    $user = [
+      'avatar' => $result['path'],
+    ];
+
+    $this->userModel->update($id, $user);
+    return redirect()->back()->with('success', '头像上传成功~');
}
```

上传头像测试一下，发现头像已经裁剪为 512x512 了

![](9.1.png)



## 提交

```shell
$ git add -A
$ git commit -m "图像裁剪"
```

