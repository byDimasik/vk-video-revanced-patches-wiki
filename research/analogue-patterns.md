# Analogue Ad-Blocking Patterns (YouTube, TikTok, Instagram) & ReVanced v22 DSL

This document details the mechanics of ReVanced Patcher v22.0.1 Kotlin DSL and documents concrete bytecode/resource ad-interception patterns analyzed from upstream YouTube, TikTok, and Instagram patches.

---

## 1. ReVanced Patcher v22 DSL Mechanics

ReVanced Patcher v22 (`v22.0.1`) introduces declarative Kotlin DSL builders alongside class definitions under `app.revanced.patcher.*`.

### Exact Package Imports

```kotlin
// DSL builders & Patch models
import app.revanced.patcher.patch.bytecodePatch
import app.revanced.patcher.patch.resourcePatch
import app.revanced.patcher.patch.BytecodePatch
import app.revanced.patcher.patch.ResourcePatch
import app.revanced.patcher.patch.annotation.Patch
import app.revanced.patcher.annotation.Name
import app.revanced.patcher.annotation.Description
import app.revanced.patcher.annotation.Version
import app.revanced.patcher.annotation.CompatiblePackage

// Fingerprint DSL & models
import app.revanced.patcher.fingerprint.method.fingerprint
import app.revanced.patcher.fingerprint.method.impl.MethodFingerprint
import app.revanced.patcher.fingerprint.method.annotation.Fuzzy

// Contexts & Bytecode Mutation
import app.revanced.patcher.patch.BytecodeContext
import app.revanced.patcher.patch.ResourceContext
import app.revanced.patcher.util.smali.SmaliCompiler
import app.revanced.patcher.extensions.Mutators.addInstructions
import app.revanced.patcher.extensions.Mutators.replaceInstruction

// Dexlib2 Opcodes and Instructions
import com.android.tools.smali.dexlib2.Opcode
import com.android.tools.smali.dexlib2.iface.instruction.Instruction
```

---

### DSL Structures & Syntax

ReVanced Patcher v22 supports both lower-camelCase builder functions (`bytecodePatch`, `resourcePatch`, `fingerprint`) and class-based inheritances (`BytecodePatch`, `ResourcePatch`, `MethodFingerprint`).

#### 1. `bytecodePatch` DSL Function & `BytecodePatch` Class

```kotlin
// Functional DSL Builder
val sampleBytecodePatch = bytecodePatch(
    name = "sample-bytecode-patch",
    description = "Mutates Dalvik bytecode instructions at build time.",
    version = "0.0.1",
    compatiblePackages = setOf("com.vk.vkvideo")
) {
    execute { context ->
        val match = sampleFingerprint.result ?: return@execute
        val method = match.mutableMethod
        
        // Mutate method bytecode
        method.replaceInstruction(0, "return-void")
    }
}

// Class-based implementation
@Patch
@Name("sample-bytecode-patch")
@Description("Mutates Dalvik bytecode instructions at build time.")
@Version("0.0.1")
@CompatiblePackage("com.vk.vkvideo")
class SampleBytecodePatch : BytecodePatch(
    fingerprints = setOf(SampleFingerprint)
) {
    override fun execute(context: BytecodeContext) {
        val match = SampleFingerprint.result ?: return
        val method = match.mutableMethod
        method.replaceInstruction(0, "return-void")
    }
}
```

#### 2. `resourcePatch` DSL Function & `ResourcePatch` Class

```kotlin
// Functional DSL Builder
val sampleResourcePatch = resourcePatch(
    name = "sample-resource-patch",
    description = "Modifies XML layouts and resource tables.",
    version = "0.0.1",
    compatiblePackages = setOf("com.vk.vkvideo")
) {
    execute { context ->
        val resId = context.resourceTable.getResourceId("id/ad_container", "com.vk.vkvideo")
        context.xmlFiles.forEach { xmlFile ->
            if (xmlFile.path.endsWith("res/layout/player_overlay.xml")) {
                // Mutate XML nodes
            }
        }
    }
}

// Class-based implementation
@Patch
@Name("sample-resource-patch")
@Description("Modifies XML layouts and resource tables.")
@Version("0.0.1")
@CompatiblePackage("com.vk.vkvideo")
class SampleResourcePatch : ResourcePatch() {
    override fun execute(context: ResourceContext) {
        val resId = context.resourceTable.getResourceId("id/ad_container", "com.vk.vkvideo")
    }
}
```

#### 3. `fingerprint` DSL Function & `MethodFingerprint` Class

```kotlin
// Functional DSL Builder
val sampleFingerprint = fingerprint(
    returnType = "V",
    parameters = listOf("Ljava/lang/String;", "I"),
    opcodes = listOf(
        Opcode.INVOKE_VIRTUAL,
        Opcode.CONST_4,
        Opcode.RETURN_VOID
    ),
    strings = listOf("ad_trigger_url")
)

// Class-based implementation
object SampleFingerprint : MethodFingerprint(
    returnType = "V",
    parameters = listOf("Ljava/lang/String;", "I"),
    opcodes = listOf(
        Opcode.INVOKE_VIRTUAL,
        Opcode.CONST_4,
        Opcode.RETURN_VOID
    ),
    strings = listOf("ad_trigger_url")
)
```

---

## 2. Concrete Ad-Interception Patterns

### Pattern 1: Video Player VAST / Mid-Roll Early-Return Short-Circuiting (YouTube Pattern)

In video player architectures (like YouTube VAST/VMAP or VK Video ad loaders), ad request methods trigger network fetches or instantiations for video ads. Short-circuiting these methods by injecting an early `return-void` or `return null` prevents ad loading entirely without breaking player execution.

#### ReVanced v22 Kotlin DSL Implementation

```kotlin
object AdLoaderFingerprint : MethodFingerprint(
    returnType = "V",
    parameters = listOf("Lcom/vk/video/ad/AdRequest;"),
    opcodes = listOf(
        Opcode.IGET_OBJECT,
        Opcode.INVOKE_VIRTUAL,
        Opcode.RETURN_VOID
    ),
    strings = listOf("request_vast_ad")
)

@Patch
@Name("video-ad-short-circuit")
@Description("Short-circuits player VAST ad loading methods.")
class VideoAdShortCircuitPatch : BytecodePatch(
    fingerprints = setOf(AdLoaderFingerprint)
) {
    override fun execute(context: BytecodeContext) {
        val match = AdLoaderFingerprint.result ?: return
        val method = match.mutableMethod

        // Insert return-void at index 0 to exit method immediately
        method.replaceInstruction(0, "return-void")
    }
}
```

#### Smali Bytecode Comparison

**Original Smali:**
```smali
.method public loadVastAd(Lcom/vk/video/ad/AdRequest;)V
    .registers 3

    iget-object v0, p0, Lcom/vk/video/ad/AdRequestLoader;->adClient:Lcom/vk/video/ad/AdClient;
    invoke-virtual {v0, p1}, Lcom/vk/video/ad/AdClient;->fetchAd(Lcom/vk/video/ad/AdRequest;)V
    return-void
.end method
```

**Patched Smali (Short-Circuited):**
```smali
.method public loadVastAd(Lcom/vk/video/ad/AdRequest;)V
    .registers 3

    return-void
    iget-object v0, p0, Lcom/vk/video/ad/AdRequestLoader;->adClient:Lcom/vk/video/ad/AdClient;
    invoke-virtual {v0, p1}, Lcom/vk/video/ad/AdClient;->fetchAd(Lcom/vk/video/ad/AdRequest;)V
    return-void
.end method
```

---

### Pattern 2: Feed / JSON Parser List Filtering via Injected Extension (TikTok / Instagram Pattern)

Feed-based apps (TikTok, Instagram, VK Video) return sponsored posts mixed inside normal feed response arrays. Rather than attempting complex opcode rewrites inside obfuscated loop logic, bytecode is patched to route the parsed `List` through an injected standalone Kotlin/Java Extension helper method (`filterAdItems`).

#### ReVanced v22 Kotlin DSL Implementation

```kotlin
object FeedParserFingerprint : MethodFingerprint(
    returnType = "Ljava/util/List;",
    parameters = listOf("Lorg/json/JSONObject;"),
    opcodes = listOf(
        Opcode.INVOKE_VIRTUAL,
        Opcode.MOVE_RESULT_OBJECT,
        Opcode.RETURN_OBJECT
    ),
    strings = listOf("items", "ads")
)

@Patch
@Name("feed-ad-filter")
@Description("Filters out ad items from feed API responses via injected extension.")
class FeedAdFilterPatch : BytecodePatch(
    fingerprints = setOf(FeedParserFingerprint)
) {
    override fun execute(context: BytecodeContext) {
        val match = FeedParserFingerprint.result ?: return
        val method = match.mutableMethod

        // Locate return-object index
        val returnIndex = method.implementation?.instructions?.indexOfLast { 
            it.opcode == Opcode.RETURN_OBJECT 
        } ?: return

        // Intercept return register v0 and pass through extension filter
        method.addInstruction(
            returnIndex,
            "invoke-static {v0}, Lcom/vk/video/patches/Extension;->filterAdItems(Ljava/util/List;)Ljava/util/List;"
        )
        method.addInstruction(
            returnIndex + 1,
            "move-result-object v0"
        )
    }
}
```

#### Injected Kotlin Extension Helper (`Extension.kt`)

```kotlin
package com.vk.video.patches

object Extension {
    @JvmStatic
    fun filterAdItems(items: List<Any>): List<Any> {
        return items.filterNot { item ->
            runCatching {
                val field = item.javaClass.getDeclaredField("isSponsored").apply { isAccessible = true }
                field.getBoolean(item)
            }.getOrDefault(false)
        }
    }
}
```

#### Smali Bytecode Comparison

**Original Smali:**
```smali
.method public parseFeed(Lorg/json/JSONObject;)Ljava/util/List;
    .registers 3

    invoke-virtual {p1, "items"}, Lorg/json/JSONObject;->getJSONArray(Ljava/lang/String;)Lorg/json/JSONArray;
    move-result-object v0
    invoke-static {v0}, Lcom/vk/video/model/Parser;->parseItems(Lorg/json/JSONArray;)Ljava/util/List;
    move-result-object v0
    return-object v0
.end method
```

**Patched Smali (Extension Injected):**
```smali
.method public parseFeed(Lorg/json/JSONObject;)Ljava/util/List;
    .registers 3

    invoke-virtual {p1, "items"}, Lorg/json/JSONObject;->getJSONArray(Ljava/lang/String;)Lorg/json/JSONArray;
    move-result-object v0
    invoke-static {v0}, Lcom/vk/video/model/Parser;->parseItems(Lorg/json/JSONArray;)Ljava/util/List;
    move-result-object v0
    invoke-static {v0}, Lcom/vk/video/patches/Extension;->filterAdItems(Ljava/util/List;)Ljava/util/List;
    move-result-object v0
    return-object v0
.end method
```

---

### Pattern 3: Resource ID Lookup and View / Litho Visibility Suppression

When UI components (such as Android View containers or Facebook Litho ComponentTree nodes) cannot be cleanly removed at the data layer, their layout visibility is suppressed or their Component layout creation returns `null` or empty Component nodes.

#### ReVanced v22 Kotlin DSL Implementation for Standard Views

```kotlin
object AdViewBinderFingerprint : MethodFingerprint(
    returnType = "V",
    parameters = listOf("Landroid/view/View;"),
    opcodes = listOf(
        Opcode.INVOKE_VIRTUAL,
        Opcode.RETURN_VOID
    ),
    strings = listOf("ad_banner_holder")
)

@Patch
@Name("suppress-ad-view-visibility")
@Description("Forces Ad View containers to View.GONE (0x8).")
class SuppressAdViewVisibilityPatch : BytecodePatch(
    fingerprints = setOf(AdViewBinderFingerprint)
) {
    override fun execute(context: BytecodeContext) {
        val match = AdViewBinderFingerprint.result ?: return
        val method = match.mutableMethod

        // Inject setVisibility(View.GONE) -> 0x8 on register p1 (v1)
        val instructions = listOf(
            "const/4 v0, 0x8",
            "invoke-virtual {p1, v0}, Landroid/view/View;->setVisibility(I)"
        )
        
        method.addInstructions(0, instructions)
    }
}
```

#### ReVanced v22 Kotlin DSL Implementation for Litho Components

Facebook Litho components (used heavily in Facebook, Instagram, VK Video feeds) create component layouts via static `onCreateLayout(Lcom/facebook/litho/ComponentContext;)Lcom/facebook/litho/Component;` methods. Returning `null` early prevents component layout rendering.

```kotlin
object LithoAdComponentFingerprint : MethodFingerprint(
    returnType = "Lcom/facebook/litho/Component;",
    parameters = listOf("Lcom/facebook/litho/ComponentContext;"),
    opcodes = listOf(
        Opcode.INVOKE_STATIC,
        Opcode.MOVE_RESULT_OBJECT,
        Opcode.RETURN_OBJECT
    ),
    strings = listOf("LithoAdBannerComponent")
)

@Patch
@Name("suppress-litho-ad-component")
@Description("Short-circuits Litho ad component creation to return null.")
class SuppressLithoAdComponentPatch : BytecodePatch(
    fingerprints = setOf(LithoAdComponentFingerprint)
) {
    override fun execute(context: BytecodeContext) {
        val match = LithoAdComponentFingerprint.result ?: return
        val method = match.mutableMethod

        // Return null at instruction index 0 to suppress Litho component creation
        val instructions = listOf(
            "const/4 v0, 0x0",
            "return-object v0"
        )
        method.replaceInstruction(0, instructions[0])
        method.addInstruction(1, instructions[1])
    }
}
```

#### Resource ID Patch Implementation

```kotlin
@Patch
@Name("collapse-ad-layout-dimensions")
@Description("Sets ad container height and width to 0dp in resources.")
class CollapseAdLayoutPatch : ResourcePatch() {
    override fun execute(context: ResourceContext) {
        val resId = context.resourceTable.getResourceId("id/ad_banner_container", "com.vk.vkvideo")
        
        context.xmlFiles.filter { it.path.endsWith(".xml") }.forEach { file ->
            val nodes = file.document.getElementsByTagName("View")
            for (i in 0 until nodes.length) {
                val node = nodes.item(i)
                val idAttr = node.attributes?.getNamedItem("android:id")
                if (idAttr?.nodeValue?.contains("ad_banner_container") == true) {
                    node.attributes?.getNamedItem("android:layout_height")?.nodeValue = "0dp"
                    node.attributes?.getNamedItem("android:layout_width")?.nodeValue = "0dp"
                    node.attributes?.getNamedItem("android:visibility")?.nodeValue = "gone"
                }
            }
        }
    }
}
```

#### Smali Bytecode Comparison (Litho Component Short-Circuit)

**Original Smali:**
```smali
.method public static onCreateLayout(Lcom/facebook/litho/ComponentContext;)Lcom/facebook/litho/Component;
    .registers 2

    invoke-static {p0}, Lcom/vk/video/ui/ad/LithoAdSpec;->createAdLayout(Lcom/facebook/litho/ComponentContext;)Lcom/facebook/litho/Component;
    move-result-object v0
    return-object v0
.end method
```

**Patched Smali (Litho Component Early-Return Null):**
```smali
.method public static onCreateLayout(Lcom/facebook/litho/ComponentContext;)Lcom/facebook/litho/Component;
    .registers 2

    const/4 v0, 0x0
    return-object v0
    invoke-static {p0}, Lcom/vk/video/ui/ad/LithoAdSpec;->createAdLayout(Lcom/facebook/litho/ComponentContext;)Lcom/facebook/litho/Component;
    move-result-object v0
    return-object v0
.end method
```

---

## 3. ReVanced v22 JVM Unit-Testing Pattern

ReVanced v22 enables headless JVM unit testing for fingerprints and bytecode instruction mutations without requiring a physical Android device or emulator. Synthetic Smali code is compiled into an in-memory DEX file using `SmaliCompiler`, passed to `BytecodeContext`, and verified with JUnit 5.

### Test Fixture Example (`VideoAdShortCircuitPatchTest.kt`)

```kotlin
package com.vk.video.patches

import app.revanced.patcher.patch.BytecodeContext
import app.revanced.patcher.util.smali.SmaliCompiler
import com.android.tools.smali.dexlib2.Opcode
import org.junit.jupiter.api.Assertions.assertEquals
import org.junit.jupiter.api.Assertions.assertTrue
import org.junit.jupiter.api.Test

class VideoAdShortCircuitPatchTest {

    private val syntheticSmali = """
        .class public Lcom/vk/video/ad/AdRequestLoader;
        .super Ljava/lang/Object;

        .method public loadVastAd(Lcom/vk/video/ad/AdRequest;)V
            .registers 3
            .line 10
            const-string v0, "request_vast_ad"
            iget-object v0, p0, Lcom/vk/video/ad/AdRequestLoader;->adClient:Lcom/vk/video/ad/AdClient;
            invoke-virtual {v0, p1}, Lcom/vk/video/ad/AdClient;->fetchAd(Lcom/vk/video/ad/AdRequest;)V
            return-void
        .end method
    """.trimIndent()

    @Test
    fun `verify fingerprint matching and early return mutation`() {
        // Compile synthetic smali into memory DEX
        val dexFile = SmaliCompiler.compile(syntheticSmali)
        val context = BytecodeContext(setOf(dexFile))

        // Execute patch against synthetic context
        val patch = VideoAdShortCircuitPatch()
        patch.execute(context)

        // Verify fingerprint matched
        assertTrue(AdLoaderFingerprint.result != null, "AdLoaderFingerprint must match synthetic smali")

        // Assert first instruction is Opcode.RETURN_VOID
        val mutatedMethod = AdLoaderFingerprint.result!!.mutableMethod
        val firstInstruction = mutatedMethod.implementation!!.instructions.first()
        
        assertEquals(Opcode.RETURN_VOID, firstInstruction.opcode, "First opcode must be RETURN_VOID")
    }
}
```

---

## 4. Required Gradle Scaffolding Files

To build ReVanced v22 patches with Gradle, three standard configuration files are required. Direct Smali/dexlib2 dependencies are declared so `com.android.tools.smali.dexlib2.Opcode` compiles cleanly on consumer classpaths.

### 1. `settings.gradle.kts`

```kotlin
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://repo.clojars.org") }
    }
}

rootProject.name = "vk-video-revanced-patches"
include(":patches")
```

### 2. `gradle/libs.versions.toml`

```toml
[versions]
kotlin = "1.9.22"
revanced-patcher = "22.0.1"
smali = "3.0.3"
junit = "5.10.1"

[libraries]
revanced-patcher = { group = "app.revanced", name = "revanced-patcher", version.ref = "revanced-patcher" }
smali-dexlib2 = { group = "com.android.tools.smali", name = "smali-dexlib2", version.ref = "smali" }
smali = { group = "com.android.tools.smali", name = "smali", version.ref = "smali" }
junit-jupiter-api = { group = "org.junit.jupiter", name = "junit-jupiter-api", version.ref = "junit" }
junit-jupiter-engine = { group = "org.junit.jupiter", name = "junit-jupiter-engine", version.ref = "junit" }

[plugins]
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
```

### 3. `patches/build.gradle.kts`

```kotlin
plugins {
    alias(libs.plugins.kotlin.jvm)
}

group = "com.vk.video"
version = "0.0.1"

dependencies {
    implementation(libs.revanced.patcher)
    implementation(libs.smali.dexlib2)
    implementation(libs.smali)

    testImplementation(libs.junit.jupiter.api)
    testRuntimeOnly(libs.junit.jupiter.engine)
}

tasks.test {
    useJUnitPlatform()
}
```
