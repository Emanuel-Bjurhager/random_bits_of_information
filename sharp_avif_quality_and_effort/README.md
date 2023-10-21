I am currently writing some code to serve images over web. I am converting my jpeg images to avif to save bandwidth using [Sharp](https://www.npmjs.com/package/sharp).

The question I had was: how does the quality and effort parameter of Sharp affect the output image? I want a decent-looking image, that is as small as possible, but I also have strict performance requirements, meaning that the conversion has to go fast. After some searching online, I did not find any good results about this use case, so I did a small experiment, and I will collect the results here.


For this experiment, I will use 2 nested for loops to test different combinations of quality and effort. The quality parameter will go from 10 to 90, and increment by 10. And the effort parameter will go from 0 to 9, incrementing by 1. The execution time and image size will be measured. The visual quality of the image will be inspected visually on a 13 inch MacBook Pro 2015 retina screen. The code was run on a MacBook Pro, early 2015 i5. The computer was on plugged in. No other work was done on the computer throughout the duration of the experiment. The execution time was measured over 3 image conversions to get a more reliable measurement. The original image was a 7.4 MB, 4912 × 3264 px, jpeg image.

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

</details>


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
