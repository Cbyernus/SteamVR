Provides a single frame's timing information to the app


Always set 'size' to sizeof(Compositor_FrameTiming) for proper versioning and avoid memory corruption.  In C# use:

	(uint)System.Runtime.InteropServices.Marshal.SizeOf(typeof(Compositor_FrameTiming));

### Properties ###

**frameStart** - Time reference for each frame.  This won't necessarily be evenly spaced.

**frameVSync** - Time from frameStart until the next VSync.  Using (current.frameStart + current.frameVSync) - (previous.frameStart + previous.frameVSync) should be a reliable way to measure frame intervals.

**droppedFrames** - Number of droppedFrames as reported by DXGI's GetFrameStatistics since the previous frame.  This is a delta of PresentRefreshCount since the last time we called it.  It is stored unsigned, but has been known to go negative, so should be cast to an int before using.

**frameIndex** - Incremented each frame.  Can be used to call [GetFrameTiming](https://github.com/ValveSoftware/openvr/wiki/IVRCompositor::GetFrameTiming) at a lower frequency than it's being updated to iterate through the history until you see a frame you recognize.

**pose** - Hmd pose used to render this frame.

    struct Compositor_FrameTiming
    {
        uint32_t size; // sizeof(Compositor_FrameTiming)
        double frameStart;
        float frameVSync; // seconds from frame start
        uint32_t droppedFrames;
        uint32_t frameIndex;
        vr::TrackedDevicePose_t pose;
    };

---

    struct Compositor_FrameTiming
    {
	    uint32_t m_nSize; // Set to sizeof( Compositor_FrameTiming )
	    uint32_t m_nFrameIndex;
	    uint32_t m_nNumFramePresents; // number of times this frame was presented
	    uint32_t m_nNumMisPresented; // number of times this frame was presented on a vsync other than it was originally predicted to
	    uint32_t m_nNumDroppedFrames; // number of additional times previous frame was scanned out
	    uint32_t m_nReprojectionFlags;

	    /** Absolute time reference for comparing frames.  This aligns with the vsync that running start is relative to. */
	    double m_flSystemTimeInSeconds;

	    /** These times may include work from other processes due to OS scheduling.
	    * The fewer packets of work these are broken up into, the less likely this will happen.
	    * GPU work can be broken up by calling Flush.  This can sometimes be useful to get the GPU started
	    * processing that work earlier in the frame. */
	    float m_flPreSubmitGpuMs; // time spent rendering the scene (gpu work submitted between WaitGetPoses and second Submit)
	    float m_flPostSubmitGpuMs; // additional time spent rendering by application (e.g. companion window)
	    float m_flTotalRenderGpuMs; // time between work submitted immediately after present (ideally vsync) until the end of compositor submitted work
	    float m_flCompositorRenderGpuMs; // time spend performing distortion correction, rendering chaperone, overlays, etc.
	    float m_flCompositorRenderCpuMs; // time spent on cpu submitting the above work for this frame
	    float m_flCompositorIdleCpuMs; // time spent waiting for running start (application could have used this much more time)

	    /** Miscellaneous measured intervals. */
	    float m_flClientFrameIntervalMs; // time between calls to WaitGetPoses
	    float m_flPresentCallCpuMs; // time blocked on call to present (usually 0.0, but can go long)
	    float m_flWaitForPresentCpuMs; // time spent spin-waiting for frame index to change (not near-zero indicates wait object failure)
	    float m_flSubmitFrameMs; // time spent in IVRCompositor::Submit (not near-zero indicates driver issue)

	    /** The following are all relative to this frame's SystemTimeInSeconds */
	    float m_flWaitGetPosesCalledMs;
	    float m_flNewPosesReadyMs;
	    float m_flNewFrameReadyMs; // second call to IVRCompositor::Submit
	    float m_flCompositorUpdateStartMs;
	    float m_flCompositorUpdateEndMs;
	    float m_flCompositorRenderStartMs;

	    vr::TrackedDevicePose_t m_HmdPose; // pose used by app to render this frame

	    uint32_t m_nNumVSyncsReadyForUse;
	    uint32_t m_nNumVSyncsToFirstView;
    };