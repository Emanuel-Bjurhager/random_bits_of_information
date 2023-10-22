I am currently writing some code to serve images over web. I am converting my jpeg images to avif to save bandwidth using [Sharp](https://www.npmjs.com/package/sharp).

The question I had was: how does the quality and effort parameter of Sharp affect the output image? I want a decent-looking image, that is as small as possible, but I also have strict performance requirements, meaning that the conversion has to go fast. After some searching online, I did not find any good results about this use case, so I did a small experiment, and I will collect the results here.


For this experiment, I will use 2 nested for loops to test different combinations of quality and effort. The quality parameter will go from 10 to 90, and increment by 10. And the effort parameter will go from 0 to 9, incrementing by 1. The execution time and image size will be measured. The visual quality of the image will be inspected visually on a 13 inch MacBook Pro 2015 retina screen. The code was run on a MacBook Pro, early 2015 i5. The computer was on plugged in. No other work was done on the computer throughout the duration of the experiment. The execution time was measured over 3 image conversions to get a more reliable measurement. The original image was a 7.4 MB, 4912 × 3264 px, jpeg image. The final image is 2048 × 1361 px.

There are some sources of errors in this experiment that must be taken into consideration:
 - The image is saved to the disk, therefore the execution time is also affected by disk speed. In my use case, I also want to take this into consideration, as disk writes are expensive. Since the computer I am running this test on has an SSD, the write speed should be fairly consistent for similar sized images.
 - The execution time can be affected by architecture and operating system. It is possible that another architecture would produce other results. Therefore, some margin of error must be considered. For example, if one quality/effort combination is 5% faster on this machine, it might not be faster on another machine.
 - Other image input formats, size, etc. might produce different result. I am doing this experiment with a typical image of my particular use case.

<details>
  <summary><i>The code:</i></summary>
  
```javascript
console.log("Quality,Effort,ExecutionTime (ms),size (kB)");
for (let effort = 0; effort <= 9; effort++) {
  for (let quality = 10; quality <= 90; quality += 10) {
    let startTime = Date.now();

    for (let i = 0; i < 3; i++) {
      await sharp(file.buffer)
        .resize(width, height)
        .avif({
          quality: quality, // quality, integer 1-100 default 50
          effort: effort, // CPU effort, between 0 (fastest) and 9 (slowest). Default 4
        })
        .toFile(
          outputPath +
            "/" +
            "effort_" +
            String(effort) +
            "_quality_" +
            String(quality) +
            ".avif"
        );
    }

    // Get file stats to retrieve the size
    const stats = fs.statSync(
      outputPath +
        "/" +
        "effort_" +
        String(effort) +
        "_quality_" +
        String(quality) +
        ".avif"
    );

    let executionTime = Date.now() - startTime;
    console.log(
      `${quality},${effort},${executionTime / 3},${stats.size / 1000}`
    );
  }
}
```
</details>

<details>
  <summary><i>The effect of the effort parameter:</i></summary>

##### Overview:

The results show that the effort has a dramatic effect on the execution time, however; the size is not affected by effort.

| Effort | Average execution time (ms) | Execution time stdev  | Average size (kB) | Average size stdev |
| :---:  |             :---:           |                 :---: |     :---:         |    :---:           |
| 0      |                        454  | 104                   |  522              |      436           |
| 1      |                        904  | 223                   |  493              |      408           |  
| 2      |                       1 229 | 289                   |  493              |      407           |
| 3      |                       1 768 | 309                   |  515              |      416           |
| 4      |                       7 460 | 2 766                 |  508              |      407           |
| 5      |                       10 910| 3 670                 |  508              |      407           |
| 6      |                       17 894| 6 145                 |  508              |      409           |
| 7      |                       12 412| 8 739                 |  507              |      408           |
| 8      |                       29 189| 10 084                |  508              |      408           |
| 9      |                       56 273| 9 781                 |  514              |      412           |




##### Graph of average execution time:

In this figure, it can be seen that the execution time increases exponentially with effort. Efforts below 4 are reasonably quick, but after that, the execution time quickly gets very large.

<img width="672" alt="graph_of_average_execution_time" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/bbdf6faf-242a-4074-93ce-9979f0f48a3b">


##### The effect of effort on image quality:

Since effort has no effect on image size (this is true for any quality, not just the average case), that raises the question, how the quality is affected by effort? To compare this, and make the difference more obvious, a small portion of the image is displayed from every effort at quality 20.

<details>
  <summary><i>Images:</i></summary>

###### Effort 0:
<img width="366" alt="effort0_quality20" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/14fb6502-d346-4b70-ba8d-914c26e70cf8">

###### Effort 1:
<img width="368" alt="effort1_quality20" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/877051ce-dfe5-40d3-ae97-fca25682c8cf">

###### Effort 2:
<img width="367" alt="effort2_quality20" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/caedddaa-9c6a-4cc4-8161-ee51b0197bb0">

###### Effort 3:
<img width="367" alt="effort3_quality20" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/c76de8f6-4c2d-4468-a238-395872fd23a2">

###### Effort 4:
<img width="369" alt="effort4_quality20" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/ad48e5f7-4c19-4d01-8dc2-c2b49668386a">

###### Effort 5:
<img width="366" alt="effort5_quality20" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/d5aedf23-bda2-4fc4-8fe8-ea6a558ccf71">

###### Effort 6:
<img width="368" alt="effort6_quality20" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/3dc032ae-2fcf-40f0-b000-2674d1bd0fad">

###### Effort 7:
<img width="368" alt="effort7_quality20" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/ab12d48b-f38c-433d-a97e-cb90ac901bea">

###### Effort 8:
<img width="367" alt="effort8_quality20" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/488dafd0-dbd0-46c7-b5ef-52afc7903d8b">

###### Effort 9:
<img width="367" alt="effort9_quality20" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/dd9ed567-c907-4e0b-b712-c0f5192e1d20">

</details>

The difference between each effort increasing from 0 to 4 is dramatic, but after that, the difference becomes much smaller, almost negligeble. This is probably because we have hit an upper limit for how good the quality can get in such a small image. To test this teory, lets compare an effort level 0 and 9 image of quality 50:

<details>
  <summary><i>Images:</i></summary>

###### Effort 0, quality 50:
<img width="367" alt="effort0_quality50" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/7c21fcc1-c886-4479-99e0-0043e7623297">


###### Effort 9, quality 50:
<img width="367" alt="effort9_quality50" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/0f68f57c-9257-498d-b0b3-f870c7cc3ecb">

</details>

Judging from these two images, it can be seen that the effort 9 image have slightly less noise, and are also a bit sharper. But the difference is not big enough to justify spending 103 times as much computing effort. Basically, the computer has to spend a lot of time trying to fit a “better” image into roughly the same space, and over some point, that is very difficult to do, with almost no improvement after a certain time.

</details>

<details>
  <summary><i>The effect of the quality parameter:</i></summary>

##### Overview:

The results show that the quality has a dramatic effect on the size, but also affects the execution time. This makes sense, as a larger image requires more processing, and is slower to write to disk.

| Quality  | Average execution time (ms) | Execution time stdev  | Average size (kB) | Average size stdev |
| :---:    |             :---:           |                 :---: |     :---:         |    :---:           |
| 10       |                      8 822  | 12 060                |  73               |      7             |
| 20       |                     10 779  | 14 627                |  128              |      8             |  
| 30       |                     12 815  | 17 347                |  193              |      8             |
| 40       |                     14 996  | 20 683                |  283              |      8             |
| 50       |                     14 280  | 15 935                |  401              |      9             |
| 60       |                     16 099  | 17 796                |  554              |      12            |
| 70       |                     17 090  | 19 120                |  706              |      15            |
| 80       |                     18 901  | 20 769                |  885              |      19            |
| 90       |                     21 612  | 23 445                |  1 331            |      21            |




##### Graph of average execution time:

In this figure, it can be seen that the execution time increases linearly with quality.

<img width="677" alt="graph_of_average_execution_time_quality" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/98236376-b88a-4803-8596-48bc304aaee2">

##### Graph of average size:
In this figure, it can be seen that the size increases exponentially with quality.

<img width="752" alt="graph_of_average_size_quality" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/4c01cf0f-a80c-49a0-83cf-feba7e6d4473">



Since quality has the most effect on the image size, this raises the question, how the visual quality is affected by the quality parameter? When does it not make sense to increase the quality anymore? To compare this, and make the difference more obvious, a small portion of the image is displayed from every quality at effort 3. Effort 3 was chosen, because, if we should choose effort 0, 1 or 2, then the CPU does not have a lot of time to fit the data into the almost fixed image size, making the quality, the most important quality parameter. By increasing the effort to 3, the CPU should have enough time to make the higher quality settings less useful, but at the expense of execution time. Let us see if this theory is correct or not:

<details>
  <summary><i>Images:</i></summary>

###### Quality 10:
<img width="366" alt="effort_3_quality_10_" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/55347a0d-27a4-40b8-8996-b6f6a63c0eea">

###### Quality 20:
<img width="369" alt="effort_3_quality_20_" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/8211ad60-787a-486f-afa2-1a1582be3fa8">

###### Quality 30:
<img width="367" alt="effort_3_quality_30_" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/ce2b9fdc-02d8-43e7-a619-9fe357a1a68a">

###### Quality 40:
<img width="367" alt="effort_3_quality_40_" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/5c22129d-d20a-460b-86e8-70a1d5d0e0e6">

###### Quality 50:
<img width="365" alt="effort_3_quality_50_" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/10427dba-b76b-467b-9d70-92df1c033b7c">

###### Quality 60:
<img width="367" alt="effort_3_quality_60_" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/36facb96-59cc-4d51-9f8b-95ff553a619e">

###### Quality 70:
<img width="366" alt="effort_3_quality_70_" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/c8773e78-499d-40b6-8380-d99e19f53a2c">

###### Quality 80:
<img width="366" alt="effort_3_quality_80_" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/4883e9c6-9330-492b-8654-40cc6ad5d094">

###### Quality 90:
<img width="366" alt="effort_3_quality_90_" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/ab34d0b1-6c8f-4e32-939c-f33615ef4ccd">



</details>

The difference between each quality increasing from 10 to 30 is dramatic, but after that, the difference becomes much smaller. In my example, I think that going from quality 40 to 50 lets more details get included in the image, but those details are mostly minor dust and noise. Therefore, I think that, going from quality 10 and stepping upwards, I think that the best image is at quality 40. There are some minor artifacts at the smooth surface, for example, some color gradients being slightly off, but overall the image looks good at quality 40 and has less noise than the image at quality 50. Comparing quality 50, with 90, the difference is negligible:


<details>
  <summary><i>Images:</i></summary>

###### Quality 40:
<img width="367" alt="effort_3_quality_40__" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/a3180765-34ca-4a27-b05f-2201697fa234">

###### Quality 50:
<img width="366" alt="effort_3_quality_90_" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/ab34d0b1-6c8f-4e32-939c-f33615ef4ccd">

###### Quality 90:
<img width="366" alt="effort_3_quality_90_" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/ab34d0b1-6c8f-4e32-939c-f33615ef4ccd">


</details>


Here, overall, quality 40 had the best quality while keeping the file size very small. However, it had some artifacts that I fear could be of issue in the future, therefore; I am going to use quality 50.


</details>


#### Summary:
In my particular use case, quality 50 turned out to be the lowest quality setting that has practically identical visual quality to the higher quality images. At quality 50, all different efforts were compared, but after effort 3, the increased effort makes practically no difference. Therefore, I am going to use quality 50, effort 3. This means that the computational resources are reduced 3.7 times, while maintaining the same image quality and size as the default settings. The size has been reduced by 7.4 times when comparing the avif to the same resolution jpeg, and 18 times compared to the original resolution jpeg.

*Quality 50, effort 3*

<img width="365" alt="quality50_effort3" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/0b434f80-18d3-45a3-9206-65c633252445">

*Quality 50, effort 4*

<img width="368" alt="quality50_effort4" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/d9618db5-b66a-4f5a-8a99-242f147699bb">

*Original image*

<img width="368" alt="original_" src="https://github.com/Emanuel-Bjurhager/random_bits_of_information/assets/71664020/85a808ba-3863-475b-8000-1be870b665d7">






If this experiment added value to your project, then consider supporting me with a cup of coffee ☕

<a href="https://www.buymeacoffee.com/emanuelb" target="_blank"><img src="https://raw.githubusercontent.com/appcraftstudio/buymeacoffee/master/Images/snapshot-bmc-button.png" alt="Buy Me A Coffee" height="41" width="174"></a>




