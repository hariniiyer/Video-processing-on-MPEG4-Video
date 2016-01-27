<pre>
<code>

#include<stdlib.h>
#include<stdio.h>
#include<sys/types.h>
#include<unistd.h>
#include<fcntl.h>

#ifdef IPP
#include <ipp.h>
#include <ippdefs.h>
#endif

typedef double FLOAT;
//typedef float FLOAT;

// Cycle Counter Code
//
// Can be replaced with ippGetCpuFreqMhz and ippGetCpuClocks
// when IPP core functions are available.
//
typedef unsigned int UINT32;
typedef unsigned long long int UINT64;
typedef unsigned char UINT8;

UINT64 startTSC = 0;
UINT64 stopTSC = 0;
UINT64 cycleCnt = 0;

#define PMC_ASM(instructions,N,buf) \
  __asm__ __volatile__ ( instructions : "=A" (buf) : "c" (N) )

#define PMC_ASM_READ_TSC(buf) \
  __asm__ __volatile__ ( "rdtsc" : "=A" (buf) )

//#define PMC_ASM_READ_PMC(N,buf) PMC_ASM("rdpmc" "\n\t" "andl $255,%%edx",N,buf)
#define PMC_ASM_READ_PMC(N,buf) PMC_ASM("rdpmc",N,buf)

#define PMC_ASM_READ_CR(N,buf) \
  __asm__ __volatile__ ( "movl %%cr" #N ",%0" : "=r" (buf) )

UINT64 readTSC(void)
{
   UINT64 ts;

   __asm__ volatile(".byte 0x0f,0x31" : "=A" (ts));
   return ts;
}


UINT64 cyclesElapsed(UINT64 stopTS, UINT64 startTS)
{
   return (stopTS - startTS);
}
						    
#ifdef IPP
void printCpuType(IppCpuType cpuType)
{
    if(cpuType==0) {printf("ippCpuUnknown 0x00\n"); return;}
    if(cpuType==0x01) {printf("ippCpuPP 0x01 Intel Pentium processor\n");return;}
    if(cpuType==0x02) {printf("ippCpuPMX 0x02 Pentium processor with MMX technology\n");return;}
    if(cpuType==0x03) {printf("ippCpuPPR 0x03 Pentium Pro processor\n");return;}
    if(cpuType==0x04) {printf("ippCpuPII 0x04 Pentium II processor\n");return;}
    if(cpuType==0x05) {printf("ippCpuPIII 0x05 Pentium III processor and Pentium III Xeon processor\n");return;}
    if(cpuType==0x06) {printf("ippCpuP4 0x06 Pentium 4 processor and Intel Xeon processor\n");return;}
    if(cpuType==0x07) {printf("ippCpuP4HT 0x07 Pentium 4 Processor with HT Technology\n");return;}
    if(cpuType==0x08) {printf("ippCpuP4HT2 0x08 Pentium 4 processor with Streaming SIMD Extensions 3\n");return;}
    if(cpuType==0x09) {printf("ippCpuCentrino 0x09 Intel Centrino mobile technology\n");return;}
    if(cpuType==0x0a) {printf("ippCpuCoreSolo 0x0a Intel Core Solo processor\n");return;}
    if(cpuType==0x0b) {printf("ippCpuCoreDuo 0x0b Intel Core Duo processor\n");return;}
    if(cpuType==0x10) {printf("ippCpuITP 0x10 Intel Itanium processor\n");return;}
    if(cpuType==0x11) {printf("ippCpuITP2 0x11 Intel Itanium 2 processor\n");return;}
    if(cpuType==0x20) {printf("ippCpuEM64T 0x20 Intel 64 Instruction Set Architecture\n");return;}
    if(cpuType==0x21) {printf("ippCpuC2D 0x21 Intel Core 2 Duo processor\n");return;}
    if(cpuType==0x22) {printf("ippCpuC2Q 0x22 Intel Core 2 Quad processor\n");return;}
    if(cpuType==0x23) {printf("ippCpuPenryn 0x23 Intel Core 2 processor with Intel SSE4.1\n");return;}
    if(cpuType==0x24) {printf("ippCpuBonnell 0x24 Intel Atom processor\n");return;}
    if(cpuType==0x25) {printf("ippCpuNehalem 0x25\n"); return;}
    if(cpuType==0x26) {printf("ippCpuNext 0x26\n"); return;}
    if(cpuType==0x40) {printf("ippCpuSSE 0x40 Processor supports Streaming SIMD Extensions instruction set\n");return;}
    if(cpuType==0x41) {printf("ippCpuSSE2 0x41 Processor supports Streaming SIMD Extensions 2 instruction set\n");return;}
    if(cpuType==0x42) {printf("ippCpuSSE3 0x42 Processor supports Streaming SIMD Extensions 3 instruction set\n");return;}
    if(cpuType==0x43) {printf("ippCpuSSSE3 0x43 Processor supports Supplemental Streaming SIMD Extension 3 instruction set\n");return;}
    if(cpuType==0x44) {printf("ippCpuSSE41 0x44 Processor supports Streaming SIMD Extensions 4.1 instruction set\n");return;}
    if(cpuType==0x45) {printf("ippCpuSSE42 0x45 Processor supports Streaming SIMD Extensions 4.2 instruction set\n");return;}
    if(cpuType==0x60) {printf("ippCpuX8664 0x60 Processor supports 64 bit extension\n");return;}
    else printf("CPU UNKNOWN\n");
    return;
}

void printCpuCapability(pStatus)
{
printf("pStatus=%d\n",(UINT32)pStatus);
if((UINT32)pStatus & ippCPUID_MMX) printf("Intel Architecture MMX technology supported\n");
if((UINT32)pStatus & ippCPUID_SSE) printf("Streaming SIMD Extensions\n");
if((UINT32)pStatus & ippCPUID_SSE2) printf("Streaming SIMD Extensions 2\n");
if((UINT32)pStatus & ippCPUID_SSE3) printf("Streaming SIMD Extensions 3\n");
if((UINT32)pStatus & ippCPUID_SSSE3) printf("Supplemental Streaming SIMD Extensions 3\n");
if((UINT32)pStatus & ippCPUID_MOVBE) printf("The processor supports MOVBE instruction\n");
if((UINT32)pStatus & ippCPUID_SSE41) printf("Streaming SIMD Extensions 4.1\n");
if((UINT32)pStatus & ippCPUID_SSE42) printf("Streaming SIMD Extensions 4.2\n");
}
#endif

// PPM Edge Enhancement Code
//
UINT8 header[22];
UINT8 R[307200];
UINT8 G[307200];
UINT8 B[307200];
UINT8 convR[307200];
UINT8 convG[307200];
UINT8 convB[307200];

int main(int argc, char *argv[])
{
    int fdin, fdout, bytesRead=0, bytesLeft, i, j;
    UINT64 microsecs=0, clksPerMicro=0, millisecs=0;
    FLOAT clkRate;
    struct timeval start, end;
    int filecount=1;
    char filename[75];
    char convfile[75];

#ifdef IPP
    IppCpuType cpuType;
    IppStatus pStatus;
    Ipp64u pFeatureMask;
    Ipp32u pCpuidInfoRegs[4];

    cpuType=ippGetCpuType();
    pStatus=ippGetCpuFeatures(&pFeatureMask, pCpuidInfoRegs);

    printCpuType(cpuType);
    printCpuCapability(pStatus);
#endif
    
    // Estimate CPU clock rate
    startTSC = readTSC();
    usleep(1000000);
    stopTSC = readTSC();
    cycleCnt = cyclesElapsed(stopTSC, startTSC);

    printf("Cycle Count=%llu\n", cycleCnt);
    clkRate = ((FLOAT)cycleCnt)/1000000.0;
    clksPerMicro=(UINT64)clkRate;
    printf("Based on usleep accuracy, CPU clk rate = %llu clks/sec,",
          cycleCnt);
    printf(" %7.1f Mhz\n", clkRate);
    
    //printf("argc = %d\n", argc);
    gettimeofday(&start, NULL);
    if(argc < 1)
    {
       printf("Usage: Sepia.ppm\n");
       exit(-1);
    }
    
    else
    {
	while(filecount<1801)
	{
	//printf("PSF:\n");
	//for(i=0;i<9;i++)
	//{
	//    printf("PSF[%d]=%lf\n", i, PSF[i]);
	//}

        //printf("Will open file %s\n", argv[1]);
	
	sprintf(filename,"Image-%d.ppm",filecount);
	sprintf(convfile,"conv-%d.ppm",filecount);

        if((fdin = open(filename, O_RDONLY, 0644)) < 0)
        {
            printf("Error opening %s\n", argv[1]);
        }
        //else
        //    printf("File opened successfully\n");

        if((fdout = open(convfile, (O_RDWR | O_CREAT), 0666)) < 0)
        {
            printf("Error opening %s\n", argv[1]);
        }
        //else
        //    printf("Output file=%s opened successfully\n", "Sepia.ppm");
    

    bytesLeft=21;

    //printf("Reading header\n");

    do
    {
        //printf("bytesRead=%d, bytesLeft=%d\n", bytesRead, bytesLeft);
        bytesRead=read(fdin, (void *)header, bytesLeft);
        bytesLeft -= bytesRead;
    } while(bytesLeft > 0);

    header[21]='\0';

    //printf("header = %s\n", header); 

    // Read RGB data
    for(i=0; i<307200; i++)
    {
        read(fdin, (void *)&R[i], 1); convR[i]=R[i];
        read(fdin, (void *)&G[i], 1); convG[i]=G[i];
        read(fdin, (void *)&B[i], 1); convB[i]=B[i];
    }

    // Start of convolution time stamp
    startTSC = readTSC();
    float r,g,b ;
    // Reference : http://stackoverflow.com/questions/4624998/convert-image-to-black-white-or-sepia-in-c-sharp
    // Skip first and last row, no neighbors to convolve with
    for(i=0; i<480; i++)
    {

        // Skip first and last column, no neighbors to convolve with
        for(j=0; j<640; j++)
        {
	    
           //Convert Red
  
            r=0;
	    r = 0.393 * (FLOAT)R[(i)*640+j] +  0.769 * (FLOAT)G[(i)*640+j] +  0.189 * (FLOAT)B[(i)*640+j];
	    if (r > 255) r = 255;
            if (r < 0) r = 0;
	    convR[(i)*640+j] = (UINT8)r;
	   
	    //Convert Green
            g=0;
	    g+= 0.349 * (FLOAT)R[(i)*640+j] + 0.686 * (FLOAT)G[(i)*640+j] + 0.168 * (FLOAT)B[(i)*640+j];
            if (g > 255) g = 255;
            if (g < 0) g = 0;
	    convG[(i)*640+j] = (UINT8)g;
	    
	    //Convert Blue
            b=0;
	    b+= 0.272 * (FLOAT)R[(i)*640+j] + 0.534 * (FLOAT)G[(i)*640+j] + 0.131 * (FLOAT)B[(i)*640+j];
            if (b > 255) b = 255;
            if (b < 0) b = 0;
	   
	    convB[(i)*640+j] = (UINT8)b;
        }
    }

    // End of convolution time stamp
   
    write(fdout, (void *)header, 21);

    // Write RGB data
    for(i=0; i<307200; i++)
    {
        write(fdout, (void *)&convR[i], 1);
        write(fdout, (void *)&convG[i], 1);
        write(fdout, (void *)&convB[i], 1);
    }


    close(fdin);
    close(fdout);
    filecount++;
    } //while loop
   }
    stopTSC = readTSC();
    cycleCnt = cyclesElapsed(stopTSC, startTSC);
    microsecs = cycleCnt/clksPerMicro;
    millisecs = microsecs/1000;

    printf("Convolution time in cycles=%llu, rate=%llu, about %llu millisecs\n",
	    cycleCnt, clksPerMicro, millisecs);

   gettimeofday(&end, NULL);
    printf("The time taken for transforming 1800 frames to Sepia is %ld msec\n", (((end.tv_sec * 1000000 + end.tv_usec) - (start.tv_sec * 1000000 + start.tv_usec)))/1000);
}

</code>
</pre>
