# Master Script 

freq_SD <- function(infile,colname,window_size = "SW to be used",includerange=FALSE,br_start=0,br_end=10,br_by= "binsizevalue",outfile=NA, write_to_file=FALSE)

{

 #reading in text file with seperators “ “ space

chr <- read.table(infile, header = FALSE)

# naming the columns of the infile

names(chr) <- c("GenomeID","BasePos","Depth")

# package required for rollmean function to work 

library(zoo) 

# finding out the rollmean with Sliding window size of 100bp 

swm <- rollmean(chr$Depth,window_size)

# forming a dataframe for the mean values 

swmdf <- data.frame(y = swm)

# naming the coumn of the dataframe

colnames(swmdf) <- c(colname)

# Determining the break sequence :

br = seq(br_start,br_end,by=br_by)

# setting the last value of the range

br[length(br)+1]= max(swmdf)

# how to get range format in the final text file 

ranges = paste(head(br,-1), br[-1], sep =" - ")

# started computing the frequency with no plot

freq = hist(swmdf[[colname]], breaks = br, include.lowest= TRUE, plot=TRUE)

# writing the output to a text file

if(includerange)

{

sample.freq = data.frame(range = ranges, y = freq$counts)

colnames(sample.freq) <- c("range", colname)

}

else

{

sample.freq = data.frame( y = freq$counts)

colnames(sample.freq) <- c(colname)    #chromosome name to be given to column#

}

if(write_to_file)

write.table(sample.freq,file = outfile, sep = " ", row.names = FALSE)



return(sample.freq)

}

# passing the whole directory as an argument and write the final output to a text file.

freq_SD_all <- function(directory, fpattern = "*.txt" ){

files=list.files(directory,pattern = fpattern)

first_run = TRUE

for(file in files){

filename = paste0(directory,"/",file)

chr = substr(file, start = 1,stop=5)

if(first_run)

{

sfreq <- freq_SD(filename, colname = chr, includerange = TRUE)

first_run=FALSE

}

else{

sfreq2 <- freq_SD(filename, colname = chr)

sfreq <-cbind(sfreq,sfreq2)

}

}

write.table(sfreq, file = "freq.txt", row.names = FALSE, sep = "\t")

}


