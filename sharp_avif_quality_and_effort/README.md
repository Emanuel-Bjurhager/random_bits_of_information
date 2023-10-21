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
