#### 进行目录检查更新时没有效果
##### 场景
使用 **Addressables.CheckForCatalogUpdates()** 进行目录更新，得到结果，根据结果判断是否需要下载资源
##### 使用代码如下
``` C#
IEnumerator CheckUpdate()
    {
        //检查是否有更新
        AsyncOperationHandle<List<string>> catalogHandle = Addressables.CheckForCatalogUpdates();
        yield return catalogHandle;

        if (catalogHandle.Status == AsyncOperationStatus.Succeeded)
        {
            if (catalogHandle.Result.Count > 0)
            {
                //直接进行资源更新
            }
        }
        else
        {
            Debug.LogError("目录检查更新失败");
            Addressables.Release(catalogHandle);
        }
    }
```
##### 出现问题
代码始终无法进入到资源更新部分，也就是说，这里的目录检查更新的结果始终为最新版本。
##### 问题原因
找了许多地方，最终发现是在**AddressableAssetSettings**中，没有勾选**Only update catalogs manually**，使得每次运行后都会自动进行目录的更新，从而检查更新时始终为最新版本
##### 解决后的代码
值得注意的是，如果目录有更新，那么在更新资源前，也必须要手动将目录进行更新，否则资源更新会是无效的
``` C#
IEnumerator CheckUpdate()
    {
        //检查是否有更新
        AsyncOperationHandle<List<string>> catalogHandle = Addressables.CheckForCatalogUpdates();
        yield return catalogHandle;

        if (catalogHandle.Status == AsyncOperationStatus.Succeeded)
        {
            //有更新则进行目录更新
            if (catalogHandle.Result.Count > 0)
            {
                //先进行目录的更新 !!!必须，不能漏
                yield return Addressables.UpdateCatalogs(catalogHandle.Result);
                //再检查资源的更新
                
            }
        }
        else
        {
            Debug.LogError("目录检查更新失败");
            Addressables.Release(catalogHandle);
        }
    }
```
#### 异步方法使用时的注意点
1.  **Addressables.CheckForCatalogUpdates()** 的返回值，其Result仅包含目录本身的信息，用于在手动更新目录时作为参数传入
2. **Addressables.GetDownloadSizeAsync()**，获取要下载的指定key的资源时，只能传入一个或多个key名称，同时在下载前，会自动比对hash文件中的相关值，有改变的key才会下载对应的资源，否则不会处理