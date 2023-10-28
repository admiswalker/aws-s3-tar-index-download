# aws-s3-tar-index-download


梱包
```bash
cd test_images; tar -cf archive.tar *; mv archive.tar ..; cd ..; # OK
#tar -cvf archive.tar -C test_images test_images/*
#tar -cf archive.tar -C test_images 01_yumekawa_animal_neko.png # OK
#tar -cf archive.tar -C test_images . # OK
#tar -cf archive.tar -C test_images * # NG

tar -tf archive.tar
```

展開
```
mkdir tmp
tar xf archive.tar -C ./tmp
```

リスト作成・表示
```
python3 tarindexer.py -i archive.tar indexfile.index
cat indexfile.index
01_yumekawa_animal_neko.png 512 161684
02_cat_sabineko.png 162816 344959
03_cat_sakura_cut_female.png 508416 219206
04_cat_sakura_cut_male.png 728576 303868
```

ファイル内の range を計算
```
512 512+161684-1 -> 512 162195
162816 162816+344959-1 -> 162816 507774
508416 508416+219206-1 -> 508416 727621
728576 728576+303868-1 -> 728576 1032443
```

S3 にアップロード
```
s3://test-s3-api-2023-1028/archive.tar
```

データのダウンロード
```
BUCKET_NAME="test-s3-api-2023-1028"
aws s3api get-object --bucket ${BUCKET_NAME} --key archive.tar --range bytes=512-162195 01_yumekawa_animal_neko.png
aws s3api get-object --bucket ${BUCKET_NAME} --key archive.tar --range bytes=162816-507774 02_cat_sabineko.png
aws s3api get-object --bucket ${BUCKET_NAME} --key archive.tar --range bytes=508416-727621 03_cat_sakura_cut_female.png
aws s3api get-object --bucket ${BUCKET_NAME} --key archive.tar --range bytes=728576-1032443 04_cat_sakura_cut_male.png
```

ファイルの一致を確認
```
cmp test_images/01_yumekawa_animal_neko.png 01_yumekawa_animal_neko.png
cmp test_images/02_cat_sabineko.png 02_cat_sabineko.png
cmp test_images/03_cat_sakura_cut_female.png 03_cat_sakura_cut_female.png
cmp test_images/04_cat_sakura_cut_male.png 04_cat_sakura_cut_male.png
```

## 参考

- [test_images](https://www.irasutoya.com/search?q=%E3%81%AD%E3%81%93)
- [tarindexer](https://github.com/devsnd/tarindexer/tree/master)
- [Indexed and seekable tar files](https://superuser.com/questions/886095/indexed-and-seekable-tar-files)
- [オブジェクトのダウンロード](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/download-objects.html)
  > オブジェクトの一部をダウンロードする必要がある場合は、AWS CLI または REST API で追加のパラメータを使用して、ダウンロードするバイトのみを指定します。詳細については、「オブジェクトの一部のダウンロード」を参照してください。
- [オブジェクトの一部のダウンロード](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/download-objects.html#download-objects-parts)

