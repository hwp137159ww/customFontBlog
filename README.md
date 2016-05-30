# customFontBlog
自定义字体使用

1.首先是最简单也普遍的做法，打包内置字符库文件：
 
把字体库文件添加到工程，如font1.ttf添加到工程，然后在工程plist添加一项Fonts provided by application，这是个数组，然后添加key item1，value就是刚才说的font1.ttf，
 
那么在工程里就可以直接使用这个字体，直接用
 
+ (UIFont *)fontWithName:(NSString *)fontName size:(CGFloat)fontSize; 即可。
 
不过需要注意的是，这个fontName不是文件名，而是里面真正的字体名。如上面的font1.ttf里面的字体是MFQingShu_Noncommercial-Regular，那就直接用
 
UIFont *font = [UIFont fontWithName:@"MFQingShu_Noncommercial-Regular" size:12];就能去到正确的字体。
 
2.但是一般来说，字体文件比较大，不该内置，而且如果都用plist预定义的方式，那肯定就没法覆盖全，导致用户不能使用更多自己喜欢的字体。所以应该用代码读取字体的方式：
 
提供字体文件路径，返回所需要字体：
-(UIFont*)customFontWithPath:(NSString*)path size:(CGFloat)size
{
    NSURL *fontUrl = [NSURL fileURLWithPath:path];
    CGDataProviderRef fontDataProvider = CGDataProviderCreateWithURL((__bridge CFURLRef)fontUrl);
    CGFontRef fontRef = CGFontCreateWithDataProvider(fontDataProvider);
    CGDataProviderRelease(fontDataProvider);
    CTFontManagerRegisterGraphicsFont(fontRef, NULL);
    NSString *fontName = CFBridgingRelease(CGFontCopyPostScriptName(fontRef));
    UIFont *font = [UIFont fontWithName:fontName size:size];
    CGFontRelease(fontRef);
    return font;
}

这样就不需要在plist设定任何东西，只需要得到字体库文件的路径，就可以取出对应的字体。
 
上面的方法对于TTF、OTF的字体都有效，但是对于TTC字体，只取出了一种字体。因为TTC字体是一个相似字体的集合体，一般是字体的组合。所以如果对字体要求比较高，所以可以用下面的方法把所有字体取出来：
 
-(NSArray*)customFontArrayWithPath:(NSString*)path size:(CGFloat)size
{
    CFStringRef fontPath = CFStringCreateWithCString(NULL, [path UTF8String], kCFStringEncodingUTF8);
    CFURLRef fontUrl = CFURLCreateWithFileSystemPath(NULL, fontPath, kCFURLPOSIXPathStyle, 0);
    CFArrayRef fontArray =CTFontManagerCreateFontDescriptorsFromURL(fontUrl);
    CTFontManagerRegisterFontsForURL(fontUrl, kCTFontManagerScopeNone, NULL);
    NSMutableArray *customFontArray = [NSMutableArray array];
    for (CFIndex i = 0 ; i < CFArrayGetCount(fontArray); i++){
        CTFontDescriptorRef  descriptor = CFArrayGetValueAtIndex(fontArray, i);
        CTFontRef fontRef = CTFontCreateWithFontDescriptor(descriptor, size, NULL);
        NSString *fontName = CFBridgingRelease(CTFontCopyName(fontRef, kCTFontPostScriptNameKey));
        UIFont *font = [UIFont fontWithName:fontName size:size];
        [customFontArray addObject:font];
    }
    
    return customFontArray;
}

不过这个方法只支持7.0以上，暂时在7.0以下没有找到方法。
 
个人看法，因为ttc里面的字体都比较相似，所以其实使用一个也足以。
 
 
 
附:(字体的介绍)
 
TTF（TrueTypeFont）是一种字库名称。TTF（TrueTypeFont）是Apple公司和Microsoft公司共同推出的字体文件格式，随着windows的流行，已经变成最常用的一种字体文件表示方式。
 
TTC字体是TrueType字体集成文件(. TTC文件)，是在一单独文件结构中包含多种字体,以便更有效地共享轮廓数据,当多种字体共享同一笔画时,TTC技术可有效地减小字体文件的大小。
TTC是几个TTF合成的字库，安装后字体列表中会看到两个以上的字体。两个字体中大部分字都一样时，可以将两种字体做成一个TTC文件，常见的TTC字体，因为共享笔划数据，所以大多这个集合中的字体区别只是字符宽度不一样，以便适应不同的版面排版要求。
