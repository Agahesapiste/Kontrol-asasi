package com.seninmodid.controlwand; // <-- Burayı kendi klasör yolunla değiştireceksin!

import net.fabricmc.api.ModInitializer;
import net.minecraft.client.Minecraft;
import net.minecraft.client.gui.components.Button;
import net.minecraft.client.gui.screens.Screen;
import net.minecraft.core.Registry;
import net.minecraft.core.registries.BuiltInRegistries;
import net.minecraft.network.chat.Component;
import net.minecraft.resources.ResourceLocation;
import net.minecraft.world.InteractionHand;
import net.minecraft.world.InteractionResultHolder;
import net.minecraft.world.entity.LivingEntity;
import net.minecraft.world.entity.player.Player;
import net.minecraft.world.item.Item;
import net.minecraft.world.item.ItemStack;
import net.minecraft.world.item.Rarity;
import net.minecraft.world.level.Level;
import net.minecraft.world.phys.AABB;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.List;

public class ControlWand implements ModInitializer {
    
    // Konsola yazı yazdırmak için gereken Logger tanımı
    public static final Logger LOGGER = LoggerFactory.getLogger("controlwand");

    // Asanın özellikleri ve sağ tıklama (use) mekaniği
    public static final Item KONTROL_ASASI = new Item(new Item.Properties().stacksTo(1).rarity(Rarity.EPIC)) {
        
        @Override
        public InteractionResultHolder<ItemStack> use(Level level, Player player, InteractionHand hand) {
            // Sadece oyuncunun ekranında (Client) çalıştırıp arayüzü açıyoruz
            if (level.isClientSide()) {
                // 100 blok etraftaki canlıları bul
                AABB taramaAlani = new AABB(
                        player.getX() - 100, player.getY() - 100, player.getZ() - 100,
                        player.getX() + 100, player.getY() + 100, player.getZ() + 100
                );
                List<LivingEntity> canlilar = level.getEntitiesOfClass(LivingEntity.class, taramaAlani);
                canlilar.remove(player); // Kendimizi listeden çıkarıyoruz

                // EKRANI AÇ: Hazırladığımız özel listeli arayüzü oyuncunun ekranına yansıtıyoruz
                Minecraft.getInstance().setScreen(new ControlWandScreen(canlilar));
            }
            return InteractionResultHolder.success(player.getItemInHand(hand));
        }
    };

    @Override
    public void onInitialize() {
        // Modun yüklendiğini belirten o ilk log yazısı
        LOGGER.info("Control Wand Modu Basariyla Yuklendi! Asa hazirlaniyor...");

        // Asayı Minecraft sistemine kaydeden kod
        Registry.register(BuiltInRegistries.ITEM, 
                new ResourceLocation("controlwand", "kontrol_asasi"), 
                KONTROL_ASASI
        );
    }

    // --- ASANIN ÖZEL EKRAN (GUI) SINIFI ---
    public static class ControlWandScreen extends Screen {
        private final List<LivingEntity> canlilar;

        public ControlWandScreen(List<LivingEntity> canlilar) {
            super(Component.literal("Canlı Kontrol Paneli"));
            this.canlilar = canlilar;
        }

        @Override
        protected void init() {
            int yPozisyonu = 40; // İlk canlının ekrandaki yüksekliği

            // Bulunan canlıları ekranda alt alta listeliyoruz
            for (LivingEntity canli : canlilar) {
                if (yPozisyonu > this.height - 40) break; // Ekran aşağıya doğru taşmasın diye sınır

                String canliAdi = canli.getType().getDescription().getString();
                
                // 1. İsim Etiketi: Canlının adını ekrana yazdırıyoruz
                this.addRenderableOnly((guiGraphics, mouseX, mouseY, partialTick) -> 
                    guiGraphics.drawString(this.font, "§e" + canliAdi, 20, yPozisyonu + 6, 0xFFFFFF)
                );

                // 2. BUTON: IŞINLANMA BUTTONU
                this.addRenderableWidget(Button.builder(Component.literal("§aIşınlan"), button -> {
                    Minecraft.getInstance().player.connection.sendUnsignedCommand(
                        "tp @s " + canli.getX() + " " + canli.getY() + " " + canli.getZ()
                    );
                    this.onClose(); // İşlem bitince ekranı kapat
                }).bounds(120, yPozisyonu, 60, 20).build());

                // 3. BUTON: ÖLDÜRME BUTTONU
                this.addRenderableWidget(Button.builder(Component.literal("§cÖldür"), button -> {
                    Minecraft.getInstance().player.connection.sendUnsignedCommand(
                        "execute as @e[uuid=" + canli.getUUID().toString() + "] run kill @s"
                    );
                    this.onClose();
                }).bounds(185, yPozisyonu, 50, 20).build());

                // 4. BUTON: İZLEME BUTTONU
                this.addRenderableWidget(Button.builder(Component.literal("§bİzle"), button -> {
                    Minecraft.getInstance().player.connection.sendUnsignedCommand(
                        "spectate @e[uuid=" + canli.getUUID().toString() + ",limit=1] @s"
                    );
                    this.onClose();
                }).bounds(240, yPozisyonu, 50, 20).build());

                yPozisyonu += 24; // Bir sonraki canlıyı 24 piksel aşağıya yazdır
            }
        }

        // Ekranın arkasını hafif karartıp "Canlı Kontrol Paneli" başlığını basar
        @Override
        public void render(net.minecraft.client.gui.GuiGraphics guiGraphics, int mouseX, int mouseY, float partialTick) {
            this.renderBackground(guiGraphics, mouseX, mouseY, partialTick);
            guiGraphics.drawCenteredString(this.font, this.title, this.width / 2, 15, 0xFFFFFF);
            super.render(guiGraphics, mouseX, mouseY, partialTick);
        }

        @Override
        public boolean isPauseScreen() {
            return false; // Ekran açıkken oyun arka planda durmasın
        }
    }
}
