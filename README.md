# Tdarr Stage Manager

This tool was designed to interface with Tdarr, and manage the staging area.
It was bourne out of a particular problem I had with level-loading the CPU workloads and GPU workloads.

## The Backstory

I have a large number of H264 files that I'm converting to HEVC. I also have a (relatively) small number of HEVC files that are inefficiently encoded, and I want to recode them in HEVC to reduce their bitrate. My hardware is quite old, the GPUs can only run H264 to HEVC. `ffmpeg` errors out when I try to recode HEVC, so those jobs should run on the CPU. 

The problem is that as soon as you enable a CPU worker, the staging area fills with jobs that "Require GPU workers"! There's no way to level-load the CPU jobs and GPU jobs. The CPUs then sit mostly idle. 

## How it works

This tool will managed the queued items in the staging area to make sure there's an equal amount of CPU work and GPU work. It does this by:

- Bumping an item in the Transcode Queue, so it gets processed next
- Re-queuing the lowest item on the Stating Area list that would bring the CPU/GPU ratio closer to 50:50
- Waiting until that item makes it to the Staging Area
	- NOTE: if you have other bumped items in your transcode queue, this waiting time could be quite long
- If it's presence in the queue brings the CPU/GPU ratio closer to 50:50, it's not kicked
- Otherwise it is kicked, and the next item below on the main Transcode Queue is bumped and the cycle repeats

### Why this?

There's no way to tell whether a job is a CPU job or a GPU job until it gets to the staging area. 
This depends on all the filters and plugins you have configured. 
The only way to ensure there's enough work for all nodes is to recursively check all files.
