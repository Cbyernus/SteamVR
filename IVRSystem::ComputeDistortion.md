`bool ComputeDistortion( Hmd_Eye eEye, float fU, float fV, DistortionCoordinates_t *pDistortionCoordinates )`

Gets the result of a single distortion value for use in a distortion map. Input UVs are in a single eye's viewport, and output UVs are for the source render target in the distortion shader.

Returns true for success. False indicates values are unsuitable.

* `eEye` - `Eye_Left` or `Eye_Right`. Determines which eye the function should return the distortion value for.
* `fU` - horizontal texture coordinate for the output pixel within the viewport
* `fV` - vertical texture coordinate for the output pixel within the viewport
* `DistortionCoordinates_t *pDistortionCoordinates` - The struct to be updated with the distortion values

ComputeDistortion returns distortion values using the following structure:
    struct DistortionCoordinates_t
    {
        float rfRed[2];
        float rfGreen[2];
        float rfBlue[2];
    };

rfRed contains the UVs for the red channel, rfGreen contains the UVs for the green channel, and rfBlue contains the UVs for the blue channel.