[]()
## Chunking flow
### Flow chart

![image.png](https://notepic-1327795028.cos.ap-chengdu.myqcloud.com/pic/202407251358457.png)
### Why only Read part\
we unuslly read 128MiB a time from the Tar to Chunking. this will prevent too much memory from being consumed.
when the ReadFileBuffer is about to reach the end, we need to seekg the next beginning of the buffer.

### Code sample
```C++
void Chunker::Chunking()
{
    bool end = false;
    uint32_t totalOffset = 0;
    while (!end)
    {
        memset((char *)readFileBuffer, 0, sizeof(uint8_t) * READ_FILE_SIZE);
        inputFile.read((char *)readFileBuffer, sizeof(uint8_t) * READ_FILE_SIZE);
        end = inputFile.eof();
        size_t len = inputFile.gcount();
        if (len == 0)
        {
            break;
        }
        localOffset = 0;
        while (((len - localOffset) >= MAXCHUNKSIZE) || (end && (localOffset < len)))
        {
            Chunk_t chunk;
            uint32_t cp = 0;
            cp = CutPoint(readFileBuffer + localOffset, len - localOffset);
            chunk.chunkPtr = (uint8_t *)malloc(cp);
            memcpy(chunk.chunkPtr, readFileBuffer + localOffset, cp);
            chunk.chunkSize = cp;
            localOffset += cp;
        }
        totalOffset += localOffset;
        inputFile.seekg(totalOffset, ios_base::beg);
    }
    return;
}
```

## Chunking methods

the different  chunking methods are reflected in the way cp is obtained. 
```C++
uint32_t cp = Chunking(); 
```

### Fixed-size

As a tradeoff between Deduplication and throughput, chunking methods typically use 8KiB as the desired average size.
```C++
uint32_t Chunker::FixedSize(){
		return FixedChunkSize;
}
```
But the fixed-size chunking will encounter a very serious boundary migration problem, which will allow the reactivity to drop.
### CDC
When the sliding hash in the window is equal to 0 with the mask that operation set in advance, it is used as a breakpoint.This is called content-defined chunking.
```C++
uint32_t Chunker::CutPointCDC(const uint8_t *src, const uint32_t len)
{
    uint32_t fp = 0;
    uint32_t i = 0;
    for (; i < len; i++)
    {
        fp = fp - Rabin[src[i-windowSize]] + Rabin[src[i]];
        if (!(fp & MASK_GEAR))
        {
            return (i + 1);
        }
    }
    return i;
};
```
The expected block size is related to the significant bit 1 of the mask. For example, if I want the block size to be 8KiB on average, then I need 13 1's in the mask.
### Gear

Gear Hash reduces the computational overhead of the rolling hash by fixing the significant bit 1 of the mask to be the least significant bit, so that gear can obtain the rolling hash value by bit operation.
```C++
uint32_t Chunker::CutPointGear(const uint8_t *src, const uint32_t len)
{
    uint32_t fp = 0;
    uint32_t i = 0;
    for (; i < len; i++)
    {
        fp = (fp >> 1) + GEAR[src[i]];
        if (!(fp & MASK_GEAR))
        {
            return (i + 1);
        }
    }
    return i;
};
```

### FastCDC
FastCDC solves two problems, one is to further improve throughput, and the other is to standardize block sizes in the 4KiB-16KiB range. It skips the first 4KiB without having to compute the hash, and its mask is also standardized to be bitwise, so that when it reaches 16KiB, it immediately gets the breakpoint, whether it meets the breakpoint condition or not. In addition, to achieve an average size of 8KiB, he uses different masks to generate breakpoints in 4K-8K and 8K-16K.

```C++
uint32_t Chunker::CutPointFastCDC(const uint8_t *src, const uint32_t len)
{
    uint32_t n;
    uint32_t fp = 0;
    uint32_t i;
    i = min(len, static_cast<uint32_t>(minChunkSize));
    n = min(normalSize, len);
    for (; i < n; i++)
    {
        fp = (fp >> 1) + GEAR[src[i]];
        if (!(fp & maskS))
        {
            return (i + 1);
        }
    }

    n = min(static_cast<uint32_t>(maxChunkSize), len);
    for (; i < n; i++)
    {
        fp = (fp >> 1) + GEAR[src[i]];
        if (!(fp & maskL))
        {
            return (i + 1);
        }
    }
    return i;
};
```

### mTar
mTar argues that the metadata head of the periodic appearance of tar reduces the Deduplication rate
```c++
void MTar(){
	translate the Tar files to mTar files;
};

FastCDC();

```

## Conclusion
The chunking method is a part of the compression system that determines the theoretical upper bound. If the obtained chunk contains extremely high boundary offset, then the best feature value selection method cannot obtain excellent results.