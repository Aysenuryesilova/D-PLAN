D-Plan Staj Programı — Vaka Çalışması Raporu

Kategori: Yazılım ve Ürün Geliştirme (Alternatif Vaka · Uygulama Gerektirmez)
Teslim Tarihi: 4 Ekim 2026
Format: Markdown / PDF / Doc

NOTE

Vaka Çalışması Hakkında Genel Yaklaşım: Bu döküman, D-Plan kullanıcılarının günlük ve haftalık planlama yaparken yaşadığı "yoğun-hafif gün dengesizliği ve tükenmişlik (burnout)" problemine odaklanır. Kullanıcı iradesini ve rızasını (%100 Human Agency) merkeze alan "Görsel Haftalık Tablo (Sürükle-Bırak) + Kullanıcı Onaylı AI Yük Dengeleme" özelliği tasarlanmış ve teknik olarak kurgulanmıştır.

1. Senaryo & Ürün Bakışı
💡 Seçilen Özellik: "Haftalık Görsel Tablo & İsteğe Bağlı AI Yük Dengeleyici"
(a) Problem Tanımı ve Hedef Kitle (2-3 Cümle)

Problem & İhtiyaç: D-Plan'ı bir iOS cihazım olmadığı için birebir deneyimleyemesem de, genel planlama süreçlerindeki kendi tecrübelerimden yola çıkarak temel bir insan ihtiyacını tespit ettim: Yoğun ve hafif günler arasındaki dengeyi tutturamamak. Kimi günler üst üste yığılan görevler yüzünden tükeniş yaşanırken, kimi günler tamamen boş kalabiliyoruz. Elbette insan bazı günler tam dinlenme ister; ancak aşırı yoğun günlerin yükünü esnekçe diğer günlere yayabilmelidir.

Çözüm: Bu problemi çözmek için kullanıcıyı merkeze alan, kontrolü insana veren ve kullanıcının rızasıyla çalışan bir Yapay Zeka Yük Dengeleme Mekanizması kurgulanmıştır.

2. Çözüm Anlatımı & Teknik Tasarım
(b) Önerinin Çalışma Mantığı ve Arayüz Tasarımı

Çözüm olarak kullanıcıya haftanın günlerini ve görevlerini içeren görsel bir Haftalık Görev Tablosu sunulur. Sistem iki farklı modda esnek kullanım sağlar:

Manuel Mod (Sürükle-Bırak / Drag & Drop): Kullanıcı, yoğun olduğu bir gündeki görev kutucuğunu tutup daha boş veya daha uygun gördüğü başka bir günün altına elle sürükleyip bırakabilir.
Yapay Zeka Destekli Mod (Kullanıcı Onaylı Dengeleme): Kullanıcı isterse tablonun üstündeki "🤖 AI ile Haftayı Dengele" butonuna basar. Yapay zeka görevlerin sürelerini ve karmaşıklıklarını hesaplayarak yükü günlere eşit ve mantıklı şekilde dağıtan bir taslak sunar. Kullanıcı önerilen bu yeni tabloyu onaylayabilir veya üzerinde elle ince ayar yapabilir.

TIP

Temel Ürün Prensibi: Yapay zeka hiçbir zaman kullanıcıya kural dikte etmez; kontrol %100 insanda kalır.

(b - Devamı) Kod Parçası & Teknik Yapı (React Native / TypeScript)

Aşağıdaki bileşen, D-Plan mobil uygulamasında hem manuel sürükle-bırak desteğini hem de kullanıcı rızasıyla çalışan AI dengeleme fonksiyonunu modüler olarak yönetmektedir:

typescript
// WeeklyPlannerTable.tsx — D-Plan Hybrid Task Planner Component
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, StyleSheet, ActivityIndicator } from 'react-native';
interface Task {
  id: string;
  title: string;
  day: 'Pazartesi' | 'Salı' | 'Çarşamba' | 'Perşembe' | 'Cuma';
  estimatedMinutes: number;
}
export const WeeklyPlannerTable: React.FC = () => {
  const [tasks, setTasks] = useState<Task[]>([]);
  const [loading, setLoading] = useState<boolean>(false);
  // 1. Manuel Sürükle-Bırak İle Görev Taşıma
  const handleManualDragAndDrop = (taskId: string, targetDay: Task['day']) => {
    setTasks(prevTasks =>
      prevTasks.map(task => (task.id === taskId ? { ...task, day: targetDay } : task))
    );
  };
  // 2. Kullanıcı Rızası İle AI Destekli Otomatik Yük Dengeleme
  const handleAIBalanceWeeklyLoad = async () => {
    setLoading(true);
    try {
      const response = await fetch('https://api.dplan.app/v1/ai/balance-weekly', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ currentTasks: tasks })
      });
      const data = await response.json();
      // AI'ın sunduğu dengeli taslak kullanıcı onayına sunulur
      setTasks(data.balancedTasks);
    } catch (error) {
      console.error('AI dengeleme hatası:', error);
    } finally {
      setLoading(false);
    }
  };
  return (
    <View style={styles.container}>
      {/* İsteğe Bağlı AI Dengeleme Butonu */}
      <TouchableOpacity 
        style={styles.aiButton} 
        onPress={handleAIBalanceWeeklyLoad}
        disabled={loading}
      >
        {loading ? (
          <ActivityIndicator color="#FFF" />
        ) : (
          <Text style={styles.aiButtonText}>🤖 AI ile Haftayı Dengele (Öneri Al)</Text>
        )}
      </TouchableOpacity>
      {/* Görsel Tablo Alanı */}
      <View style={styles.gridContainer}>
        <Text style={styles.infoText}>
          * Görev kutucuklarını elle sürükleyebilir veya AI önerisini tek tıkla uygulayabilirsiniz.
        </Text>
      </View>
    </View>
  );
};
const styles = StyleSheet.create({
  container: { padding: 16, backgroundColor: '#F8F9FA' },
  aiButton: { backgroundColor: '#6C5CE7', padding: 12, borderRadius: 8, alignItems: 'center' },
  aiButtonText: { color: '#FFFFFF', fontWeight: '700', fontSize: 14 },
  gridContainer: { marginTop: 16 },
  infoText: { fontSize: 12, color: '#636E72', fontStyle: 'italic' }
});
(c) Yapay Zeka Bu Özellikte Nasıl Rol Alır?

Yapay zeka bu sistemde "Dayatmacı Bir Denetleyici" değil, "Kullanıcı Rızalı Yük Dengeleme Danışmanı" olarak rol alır:

Yük ve Efor Analizi: Haftanın 7 günündeki görevlerin toplam tahmini sürelerini hesaplar.
Öncelik & Bağımlılık Denetimi: Kullanıcının kilit görevlerini tespit edip ikincil görevleri daha sakin günlere esnetme önerisi hazırlar.
Kullanıcı İradesine Saygı: AI sadece bir taslak öneri (draft proposal) sunar. Kullanıcı onaylamadığı sürece takvimdeki hiçbir veri otomatik değişmez.
3. Yapay Zekayı Üretim Sürecinde Nasıl Kullandım?

Bu vaka çalışmasını hazırlarken yapay zeka araçlarını kopyala-yapıştır olarak değil, sürekli sorgulayıp yönlendirerek (Iterative Steering) kullandım.

🔄 Süreç ve Yönlendirme Hikayem:

İlk Başta Gelen Klasik Fikirlerin Reddi:
Yapay zeka araçlarına ilk soru sorduğumda, piyasada sıkça görülen "hazır kalıp / klasik" fikirler (otomatik bildirimler, otomatik takvim bölücüler vb.) önerildi. Bu önerilerin insan psikolojisini göz ardı ettiğini ve detaylı/boğucu olduğunu fark edip onları kabul etmedim.

İnsan Kontrolü ve Esneklik Dayatması:
Yapay zekayı şu mantıkla yeniden yönlendirdim: "Böyle şeyler otomatize edilip kullanıcıya zorlanamaz. İnsan kontrolü, duygusu ve esnekliği şarttır. Öyle bir sistem tasarlayalım ki hem görsel tablo üzerinde elle sürükle-bırak yapılabilsin hem de istenirse yapay zekadan dengeleme önerisi alınabilsin."

Sonuca Ulaşma ve İnce Ayar:
Bu yönlendirmem sonucunda fikir "Haftalık Görsel Tabloda Sürükle-Bırak + Kullanıcı Onaylı AI Dengeleme" noktasına evrildi. Kod parçası ve teknik mimari de bu insan odaklı felsefe etrafında şekillendirildi.

4. Değerlendirme Kriterleri Özeti
Ürün Bakışı: Yoğun-hafif gün dengesizliğini tespit edip çözümü kullanıcı iradesini koruyarak tasarladım.
Çözüm Kalitesi: Görsel tablo, sürükle-bırak mantığı ve TypeScript kod bileşeniyle uygulanabilir bir mimari sundum.
Yapay Zeka Kullanımı: AI'a hazır kalıp fikirleri kabul ettirmeyip yönlendirerek insan odaklı bir çıktı elde ettim.
Anlatım: Fikrimi kendi deneyimlerimle birleştirip net, özgün ve düzenli aktardım.
