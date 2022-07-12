# AN Optimal Score for Image Quality
Users upload images of various sizes (kB, MB) and resolutions. Images with high resolution or size results in longer upload times, higher space utilization on cloud/server and high latency while trying to view or download the images. To address the above challenges, we need to find
a way to store images on the cloud/server with an optimal image quality, which strikes a fine
balance between image quality and image size.
This project helps us to find an optimal score (range 0-100) of pixel density for an image to which the input image can be dropped to, while preserving most of the image quality at the same time.

To get started, lets install **Pillow**
'$ pip install Pillow'

Open up a new Python file and import it:
'import os
from PIL import Image'

Before we dive into compressing images, let's grab a function to print the file size in a friendly format:
'def get_size_format(b, factor=1024, suffix="B"):
    """
    Scale bytes to its proper byte format
    e.g:
        1253656 => '1.20MB'
        1253656678 => '1.17GB'
    """
    for unit in ["", "K", "M", "G", "T", "P", "E", "Z"]:
        if b < factor:
            return f"{b:.2f}{unit}{suffix}"
        b /= factor
    return f"{b:.2f}Y{suffix}"'

Next, let's make our core function for compressing images:
'def compress_img(image_name, new_size_ratio=0.9, quality=90, width=None, height=None, to_jpg=True):
    # load the image to memory
    img = Image.open(image_name)
    # print the original image shape
    print("[*] Image shape:", img.size)
    # get the original image size in bytes
    image_size = os.path.getsize(image_name)
    # print the size before compression/resizing
    print("[*] Size before compression:", get_size_format(image_size))
    if new_size_ratio < 1.0:
        # if resizing ratio is below 1.0, then multiply width & height with this ratio to reduce image size
        img = img.resize((int(img.size[0] * new_size_ratio), int(img.size[1] * new_size_ratio)), Image.ANTIALIAS)
        # print new image shape
        print("[+] New Image shape:", img.size)
    elif width and height:
        # if width and height are set, resize with them instead
        img = img.resize((width, height), Image.ANTIALIAS)
        # print new image shape
        print("[+] New Image shape:", img.size)
    # split the filename and extension
    filename, ext = os.path.splitext(image_name)
    # make new filename appending _compressed to the original file name
    if to_jpg:
        # change the extension to JPEG
        new_filename = f"{filename}_compressed.jpg"
    else:
        # retain the same extension of the original image
        new_filename = f"{filename}_compressed{ext}"
    try:
        # save the image with the corresponding quality and optimize set to True
        img.save(new_filename, quality=quality, optimize=True)
    except OSError:
        # convert the image to RGB mode first
        img = img.convert("RGB")
        # save the image with the corresponding quality and optimize set to True
        img.save(new_filename, quality=quality, optimize=True)
    print("[+] New file saved:", new_filename)
    # get the new image size in bytes
    new_image_size = os.path.getsize(new_filename)
    # print the new size in a good format
    print("[+] Size after compression:", get_size_format(new_image_size))
    # calculate the saving bytes
    saving_diff = new_image_size - image_size
    # print the saving percentage
    print(f"[+] Image size change: {saving_diff/image_size*100:.2f}% of the original image size.")'

A giant function that does a lot of stuff, let's cover it in more detail:
1. First, we use the *Image.open()* method to load the image to the memory, we get the size of the image file using *os.path.getsize()* so we can later compare this size with the new generated file's size.
2. If *new_size_ratio* is set below *1.0*, then resizing is necessary. This number ranges from 0 to 1 and is multiplied by the *width* and *height* of the original image to come up with a lower resolution image. This is a suitable parameter if you want to reduce the image size further. You can also set it to *0.95* or *0.9* to reduce the image size with minimal changes to the resolution.
3. If *new_size_ratio* is *1.0*, but *width* and *height* are set, then we resize to these new *width* and *height* values, make sure they're below the original *width* and *height*.
4. If *to_jpg* is set to *True*, we change the extension of the original image to be JPEG. This will significantly reduce image size, especially for PNG images. If the conversion raises an *OSError*, converting the image format to RGB will solve the issue.
5. Finally, we use the *save()* method to write the optimized image, and we set *optimize* to *True* along with the quality passed from the function. We then get the size of the new image and compare it with the size of the original image.

Now that we have our core function, let's use *argparse* module to integrate it with the command-line arguments:
'if __name__ == "__main__":
    import argparse
    parser = argparse.ArgumentParser(description="Simple Python script for compressing and resizing images")
    parser.add_argument("image", help="Target image to compress and/or resize")
    parser.add_argument("-j", "--to-jpg", action="store_true", help="Whether to convert the image to the JPEG format")
    parser.add_argument("-q", "--quality", type=int, help="Quality ranging from a minimum of 0 (worst) to a maximum of 95 (best). Default is 90", default=90)
    parser.add_argument("-r", "--resize-ratio", type=float, help="Resizing ratio from 0 to 1, setting to 0.5 will multiply width & height of the image by 0.5. Default is 1.0", default=1.0)
    parser.add_argument("-w", "--width", type=int, help="The new width image, make sure to set it with the `height` parameter")
    parser.add_argument("-hh", "--height", type=int, help="The new height for the image, make sure to set it with the `width` parameter")
    args = parser.parse_args()
    # print the passed arguments
    print("="*50)
    print("[*] Image:", args.image)
    print("[*] To JPEG:", args.to_jpg)
    print("[*] Quality:", args.quality)
    print("[*] Resizing ratio:", args.resize_ratio)
    if args.width and args.height:
        print("[*] Width:", args.width)
        print("[*] Height:", args.height)
    print("="*50)
    i=img.size
    compress_img(args.image, args.resize_ratio, args.quality, i[0], i[1], args.to_jpg)'

**To use our script we need to type python filename.py imagename.jpg -j -q 8**



